# Black Duck SCA — MCP Server

[![PyPI - Version](https://img.shields.io/pypi/v/blackduck-sca-mcp)](https://pypi.org/project/blackduck-sca-mcp/)
[![PyPI - License](https://img.shields.io/pypi/l/blackduck-sca-mcp)](https://pypi.org/project/blackduck-sca-mcp/)
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
6. [Troubleshooting](#troubleshooting)
7. [Support](#support)

## Capabilities

These are the MCP tool areas currently available.

| Area | Available Features | Related Tool Names |
|------|--------------------|--------------------|
| Dashboard | View instance-wide security posture, activity trends, and vulnerability breakdowns | get_dashboard_summary |
| Projects | Find projects and versions, inspect BOM contents, and review project-level vulnerabilities | search_projects_versions<br>fetch_project_components<br>fetch_project_vulnerabilities |
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

```bash
claude mcp add blackduck-bdsca-mcp \
  --env BLACKDUCK_BDSCA_URL=https://<your-instance-url> \
  --env BLACKDUCK_BDSCA_TOKEN=<api-token> \
  -- uvx \
  --managed-python --python 3.13 \
  --from 'blackduck-sca-mcp' bdsca
```

### Claude Desktop/Cowork

Add via UI and make sure the config file has similar values afterward.

```json
{
    "managedMcpServers": [
    {
      "name": "blackduck-bdsca-mcp",
      "source": "user",
      "transport": "stdio",
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
| `BLACKDUCK_BDSCA_SSL_VERIFY`                                       | `true`              | Whether to verify SSL/TLS certificates for BDSCA connections             |
| `BLACKDUCK_BDSCA_LOG_LEVEL`                                        | `INFO`              | Logging level: `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`          |
| `BLACKDUCK_BDSCA_LOG_FORMAT`                                       | `colored`           | Log format: `colored`, `json`, `simple`                                 |
| `BLACKDUCK_BDSCA_LOG_FILE`                                         | *(none)*            | Path to a log file (written in addition to stderr)                      |

All three logging variables can also be set via CLI flags for the MCP command (`--log-level`, `--log-format`,
`--log-file`), which take precedence over environment variables.


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
   [Role and Permission Matrix](https://documentation.blackduck.com/bundle/bd-hub/page/UsersAndGroups/RoleMatrix.html)
   for the full list.

Common role requirements:

| Operation | Minimum required role |
|-----------|----------------------|
| Run scans | `GLOBAL_CODE_SCANNER`, `PROJECT_CODE_SCANNER`, or `PG_CODE_SCANNER` |
| Create new projects | `PROJECT_CREATOR`, `FULL_ACCESS`, or `PG_MANAGER` |
| Update vulnerability remediation | Write access to the project version |

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
- [Email](support@blackduck.com)
