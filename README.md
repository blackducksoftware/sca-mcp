# Black Duck SCA — MCP Server

[![PyPI - Version](https://img.shields.io/pypi/v/blackduck-sca-mcp)](https://pypi.org/project/blackduck-sca-mcp/)
[![PyPI - License](https://img.shields.io/pypi/l/blackduck-sca-mcp)](https://github.com/blackducksoftware/sca-mcp/blob/main/LICENSE)
[![Website](https://img.shields.io/badge/website-https%3A%2F%2Fwww.blackduck.com-blue)](https://www.blackduck.com/)
[![Email](https://img.shields.io/badge/email-support%40blackduck.com-green)](mailto:support@blackduck.com)


Connect your AI coding assistant to the Black Duck Software Composition Analysis (SCA) products.
The MCP server exposes your Black Duck SCA instance as a set of tools that any
MCP-compatible AI harness can call to perform actions such as: 
investigate your software's security posture and vulnerabilities, triage findings, 
generate vulnerability reports and SBOMs.

## Table of Contents

1. [Capabilities](#capabilities)
2. [Prerequisites](#prerequisites)
3. [Installation](#installation)
4. [Security](#security)
5. [Environment Variables](#environment-variables)
6. [First Steps With The MCP](#first-steps-with-the-mcp)
7. [Troubleshooting](#troubleshooting)
8. [Support](#support)

## Capabilities

These are the MCP tool areas currently available.

| Area | Available Features | Related Tool Names |
|------|--------------------|--------------------|
| Dashboard | View instance-wide security posture, activity trends, and vulnerability breakdowns | get_dashboard_summary |
| Projects | Find projects and versions, inspect BOM (Bill Of Materials) contents, and review project-level vulnerabilities | search_projects_versions<br>fetch_project_components<br>fetch_project_vulnerabilities<br>search_lts_projects_versions<br>fetch_lts_project_components |
| Components | Search components across projects and see where specific versions are used | search_components |
| Vulnerabilities | Search the global vulnerability dataset and update remediation/triage status | search_vulnerabilities<br>update_vulnerability_remediation |
| Policies | Check policy violation status and compliance at project version level | fetch_policy_violation_status |
| Scanning | Run scans for source, binary, container, and SBOM inputs; check scan status and results; match code snippets | scan<br>search_scans<br>get_scan_status<br>match_code_snippet |
| Reports | Generate SBOM, VEX/CSAF, and Notices reports | create_report |
| Status | Full setup and connectivity check: Python/server version, env vars, Java, Detect JAR, connection | check_status |

You can always check the full list of available tools by asking your AI assistant to list them, e.g. 
"What Black Duck SCA tools do you have access to and what are their functions?"

## Prerequisites

- **Black Duck SCA instance** with a user account and API token
- **Python 3.13 or later+**
- **[uv](https://docs.astral.sh/uv/getting-started/installation/)** — Python package manager
- **Java 11+** _(optional)_ — required only for source scanning via Detect

## Installation

### Claude Code

#### MCP only

```bash
claude mcp add blackduck-bdsca-mcp \
  --env BLACKDUCK_BDSCA_URL=https://<your-instance-url> \
  --env BLACKDUCK_BDSCA_TOKEN=<api-token> \
  -- uvx \
  --managed-python --python 3.13 \
  --from 'blackduck-sca-mcp' bdsca
```

#### Plugin

For Claude Code there is a plugin available that ships the MCP with specialized skills 
and workflows. See installation instructions at https://github.com/blackducksoftware/sca-llm-plugin 

### Claude Desktop/Cowork

Add via UI and make sure the config file has similar values afterward.

```json
{
    "managedMcpServers": [
    {
      "name": "blackduck-bdsca-mcp",
      "source": "user",
      "command": "/bin/uvx",
      "args": [
        "--managed-python",
        "--python", "3.13",
        "--from", "blackduck-sca-mcp", "bdsca"
      ],
      "env": {
        "BLACKDUCK_BDSCA_URL": "https://<your-instance-url>",
        "BLACKDUCK_BDSCA_TOKEN": "<api-token>"
      }
    }
  ]
}

```

###  VS Code

Add to `.vscode/mcp.json` in your project:

```json
{
  "servers": {
    "blackduck-bdsca-mcp": {
      "command": "uvx",
      "args": [
        "--managed-python",
        "--python", "3.13",
        "--from", "blackduck-sca-mcp", "bdsca"
      ],
      "env": {
        "BLACKDUCK_BDSCA_URL": "https://<your-instance-url>",
        "BLACKDUCK_BDSCA_TOKEN": "<api-token>"
      }
    }
  }
}
```

###  Copilot CLI

Copilot CLI currently does not implement the full  
MCP spec, it is missing handling of MCP resources. To work around that, we can expose those resources as tools 
by specifying the `BLACKDUCK_MCP_ENABLE_RESOURCES_AS_TOOLS=true` environment variable.

```bash
copilot mcp add blackduck-bdsca-mcp \
  --env BLACKDUCK_BDSCA_URL=https://<your-instance-url> \
  --env BLACKDUCK_BDSCA_TOKEN=<api-token> \
  --env BLACKDUCK_MCP_ENABLE_RESOURCES_AS_TOOLS=true \
  -- uvx \
  --managed-python --python 3.13 \
  --from 'blackduck-sca-mcp' bdsca
```

```json
{
  "servers": {
    "blackduck-bdsca-mcp": {
      "command": "uvx",
      "args": [
        "--managed-python",
        "--python", "3.13",
        "--from", "blackduck-sca-mcp", "bdsca"
      ],
      "env": {
        "BLACKDUCK_BDSCA_URL": "https://<your-instance-url>",
        "BLACKDUCK_BDSCA_TOKEN": "<api-token>"
      }
    }
  }
}
```

#### Roo Code

Use the same `.vscode/mcp.json` configuration as GitHub Copilot above.

### SSL/TLS

If your instance uses a self-signed certificate, either add it to your system's
trusted certificate store or set below for the MCP to allow it to connect:

```bash
export BLACKDUCK_BDSCA_SSL_VERIFY=false
```

## Security

API tokens inherit the full permissions of the associated user account.
If you provide a token with write access, the AI assistant can modify
data in your Black Duck SCA instance — including updating vulnerability
remediation status and policy overrides.

We recommend creating a dedicated service account with the minimum
permissions required for your use case. See the [Role and Permission Matrix](https://documentation.blackduck.com/bundle/bd-hub/page/UsersAndGroups/RoleMatrix.html) for details.


## Environment Variables

Available environment variables for configuring the MCP server:

| Variable                                                           | Default             | Description                                                             |
|--------------------------------------------------------------------|---------------------|-------------------------------------------------------------------------|
| `BLACKDUCK_BDSCA_URL`                                              | `https://localhost` | Base URL of the Black Duck BDSCA instance                               |
| `BLACKDUCK_BDSCA_TOKEN`                                            | *(required)*        | API bearer token for BDSCA authentication                               |
| `BLACKDUCK_BDSCA_SSL_VERIFY`                                       | `true`              | Whether to verify SSL/TLS certificates for BDSCA connections            |
| `BLACKDUCK_BDSCA_LOG_LEVEL`                                        | `INFO`              | Logging level: `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`          |
| `BLACKDUCK_BDSCA_LOG_FORMAT`                                       | `colored`           | Log format: `colored`, `json`, `simple`                                 |
| `BLACKDUCK_BDSCA_LOG_FILE`                                         | *(none)*            | Path to a log file (written in addition to stderr)                      |
| `BLACKDUCK_MCP_ENABLE_RESOURCES_AS_TOOLS`                          | `false`             | Whether to expose a separate tool to access the MCP resources. This is a compatibility mode for the AI harnesses that don't implement the full MCP spec. |  

All three logging variables can also be set via CLI flags for the MCP command (`--log-level`, `--log-format`,
`--log-file`), which take precedence over environment variables.


## First Steps With The MCP

To get started with the BDSCA MCP, you can ask the LLM some starter questions such as:

```
Am I connected to the BDSCA instance?
What is the status of the BDSCA instance?
```

That kind of questions will trigger the LLM to return information about the MCP server version, BDSCA server version, related environment variables configured etc.

You can then continue by asking for example:

```
Give me an overview of my BDSCA instance
Give me an overview of my security posture
```

That kind of questions will trigger the LLM to return information about BDSCA instance's amount of Projects, Components, Vulnerabilities etc. and top security risks.

The LLM will usually then suggest next steps on what Projects and Project Versions to drill into, or what Vulnerabilities should be prioritized to be remediated within which Project Versions.

### Exploring Projects, Versions, and BOMs

```
Find the Project "my-project" and list its Project Versions
Show me the BOM for my-project 1.4.0
What Vulnerabilities affect my-project 1.4.0?
```

That kind of questions will trigger the LLM to return matching Projects, the BOM with License info per Component, and the Vulnerability breakdown per severity for the given Project Version.

### Cross-project Component Lookup

```
Which Projects use the log4j-core 2.14.1?
Search for Components matching "spring-boot"
```

That kind of questions will trigger the LLM to return all Projects and Project Versions using a given Component.

### Vulnerability Search and Triage

```
Which of my Projects are affected by the CVE-2021-44228?
```

That kind of questions will trigger the LLM to return the CVE details and the list of the impacted Project Versions.

Instructing the LLM to triage will fetch Project Vulnerabilities, and finally batch-apply a Remediation status like `NOT_AFFECTED`, `MITIGATED`, or `PATCHED` with the justification and comments.

### Policy Compliance

```
What policy violations I have in my projects?
Which policies does my-project 1.4.0 violate?
```

That kind of questions will trigger the LLM to return the compliance state and details of any violated Policy Rules for the given Project Version.

### Scanning

```
Scan this <directory path> for Components and Vulnerabilities
Scan this <file path> SBOM file / container image / binary
```

That kind of questions will trigger the LLM to scan the given file(s) and return results after the Scan has finished.

### Reports

```
Generate an SBOM in CycloneDX format for my-project 1.4.0
Create a VEX/CSAF document for my-project 1.4.0
Export a Notices report for my-project 1.4.0
```

That kind of questions will trigger the LLM to create desired reports.


## Troubleshooting

### Enable debug logging to a file

The fastest way to diagnose any issue is to capture a full debug log. Add these two
environment variables to your MCP server configuration:

```
BLACKDUCK_BDSCA_LOG_LEVEL=DEBUG
BLACKDUCK_BDSCA_LOG_FILE=/tmp/bdsca-mcp.log
```

**Claude Code example** — re-add the server with logging enabled:

```bash
export BLACKDUCK_BDSCA_LOG_LEVEL=DEBUG
export BLACKDUCK_BDSCA_LOG_FILE=/tmp/bdsca-mcp.log
claude
```

**VS Code / Claude Desktop** — add to the `env` block in your `mcp.json` / config file:

```json
"env": {
  "BLACKDUCK_BDSCA_URL": "https://<your-instance-url>",
  "BLACKDUCK_BDSCA_TOKEN": "<api-token>",
  "BLACKDUCK_BDSCA_LOG_LEVEL": "DEBUG",
  "BLACKDUCK_BDSCA_LOG_FILE": "/tmp/bdsca-mcp.log"
}
```

Then run the failing operation and inspect `/tmp/bdsca-mcp.log`.

### Verify startup — check the first log lines

On a successful start the log will contain a line like:

```
INFO  BDSCA MCP server starting  version=1.2.3  bdsca_url=https://...  token=eyJh...9Qw
```

If `token=<not_set>` the `BLACKDUCK_BDSCA_TOKEN` environment variable was not passed
to the MCP process. Check your MCP client configuration and restart.

If `bdsca_url` shows `None` or is missing, `BLACKDUCK_BDSCA_URL` is also unset.

### Run the status check

Ask your AI assistant:

> "Check the Black Duck MCP server status"

This calls `check_status` and returns a full diagnostic snapshot: Python and server
version, registered tool and resource counts, all relevant environment variables
(with tokens redacted), Java availability for source scanning, Detect JAR cache state,
and a live connection test against the Black Duck instance. It is the quickest way to
confirm that everything is configured and reachable before running any other tool.

### Unable to access resources

**Symptom.** MCP calls produce an error like 

> "Fetched resource, but the following was an invalid URL: workflow-instructions://vulnerability-remediation"

This is caused when the used AI harness is not able to use resources or templated resources provided by the MCP server. To work around that, 
we can expose those resources as tools by specifying the `BLACKDUCK_MCP_ENABLE_RESOURCES_AS_TOOLS=true` environment variable.


### Token is wrong or expired

**Symptom:** `check_status` returns an error, or every tool call fails with an
authentication error.

**Steps:**

1. Open the log file and look for a line containing `Could not authenticate with token`
   or `api_call  status=401`.
2. Verify the token in your Black Duck instance:
   - Log in to your Black Duck instance.
   - Go to **User → My Access Tokens** and confirm the token is active and not expired.
3. Update the token in your MCP client configuration and restart.

> **Note:** API tokens are user-specific. If the token owner's account is disabled existing 
tokens may be invalidated.

### Insufficient permissions

**Symptom:** Scan or remediation operations fail with "Insufficient permissions".

**Steps:**

1. The log will show the specific missing roles, e.g.:
   `Missing scan access role. Required one of: GLOBAL_CODE_SCANNER, ...`
2. Ask a Black Duck administrator to assign one of the required roles to the account
   associated with your API token. See the
   [Role and Permission Matrix](https://docs.blackduck.com/r/blackduck/latest/black-duck-documentation/black-duck-user-role-matrix.html)
   for the full list.

Common role requirements:

| Operation | Minimum required role (from smallest to largest permissions) |
|-----------|--------------------------------------------------------------|
| Run Scans | `Project Code Scanner` or `Project Group Code Scanner` or `Global Code Scanner` |
| Create new Projects | `Project Administrator` or `Project Manager` or `Global Project Administrator` or `Global Project Manager` |
| Update Vulnerability Remediation | `Project Manager` or `Global Project Manager` |

### Java not found (source scanning)

**Symptom:** Source scan fails with a Java-related error, e.g.
`Java executable not found: 'java'`.

Black Duck Detect — used for source, directory, and Docker Inspector scans — requires
Java 11 or later. The MCP server searches for Java in this order:
1. `DETECT_JAVA_PATH` env var (direct path to the `java` executable)
2. `JAVA_HOME` env var
3. `java` on the system `PATH`

**Steps:**

1. Verify Java is installed: `java -version`
2. If Java is installed but not on the PATH, set the env var in your MCP config:
   ```
   DETECT_JAVA_PATH=/path/to/jdk/bin/java
   ```

### Detect JAR download blocked (source scanning)

**Symptom:** Source scan fails with `Could not obtain Detect JAR` or
`Failed to contact repository`.

The MCP server downloads the Detect JAR from `https://repo.blackduck.com` on first use. If
that host is blocked by a corporate firewall:

**Option A — Use an internal mirror:**

```
DETECT_SOURCE=https://your-internal-mirror/detect-X.Y.Z.jar
```

**Option B — Pre-download and provide the local path:**

Download the JAR manually from `repo.blackduck.com` and set:

```
DETECT_JAR_PATH=/path/to/detect-X.Y.Z.jar
```

### SSL / self-signed certificate

**Symptom:** Connection fails with a certificate verification error.

If your Black Duck instance uses a self-signed or internally-issued certificate:

```
BLACKDUCK_BDSCA_SSL_VERIFY=false
```

Or point to your CA bundle:

```
BLACKDUCK_BDSCA_SSL_VERIFY=/path/to/ca-bundle.crt
```

### Rate limiting

**Symptom:** Tool calls start returning "Rate limit exceeded" or slow down significantly
under heavy use.

The MCP server applies two independent rate limits:

| Limit | Environment variable | Default |
|-------|---------------------|---------|
| Inbound MCP requests (from the AI agent) | `BLACKDUCK_BDSCA_MCP_MAX_REQUESTS_PER_SEC` | `10` |
| Outbound API calls (to Black Duck) | `BLACKDUCK_BDSCA_API_MAX_REQUESTS_PER_SEC` | `10` |

If you have exclusive use of the Black Duck instance and need higher throughput, increase
both values. If you are sharing the instance with other users, keep the outbound limit
conservative to avoid impacting others.


## Support

- [Black Duck Website](https://www.blackduck.com/)
- [Community Portal](https://community.blackduck.com)
- [Email](mailto:support@blackduck.com)
