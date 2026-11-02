# Black Duck SCA (BDSCA) MCP Server

The BDSCA MCP server connects your AI assistant to a Black Duck Software Composition
Analysis (SCA) instance. It can investigate your security posture, browse Projects and BOMs,
triage Vulnerabilities, check Policy compliance, run scans and generate reports.

For an overview of both servers and the shared security guidance, see the
[main README](../README.md). For the Black Duck Binary Analysis server, see [BDBA](bdba.md).

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Installation](#installation)
3. [Environment Variables](#environment-variables)
4. [Authentication](#authentication)
5. [Read-only Mode](#read-only-mode)
6. [Tools](#tools)
7. [Resources](#resources)
8. [First Steps](#first-steps)
9. [Troubleshooting](#troubleshooting)

## Prerequisites

- **Black Duck SCA instance** with a user account and API token
- **Python 3.13** and **[uv](https://docs.astral.sh/uv/getting-started/installation/)**
- **Java 11+** _(optional)_ — required only for source scanning via Detect

## Installation

The server is started with the `bdsca` command from the `blackduck-sca-mcp` package. Add it
to your AI assistant as described below. The full list of settings is in
[Environment Variables](#environment-variables).

<details>

<summary>Claude Code</summary>

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

</details>

<details>

<summary>Claude Desktop (Cowork)</summary>

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

</details>

<details>

<summary>VS Code</summary>

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

</details>

<details>

<summary>Copilot CLI</summary>

Copilot CLI currently does not implement the full MCP spec, it is missing handling of MCP
resources. To work around that, we can expose those resources as tools by specifying the
`BLACKDUCK_MCP_ENABLE_RESOURCES_AS_TOOLS=true` environment variable.

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
        "BLACKDUCK_BDSCA_TOKEN": "<api-token>",
        "BLACKDUCK_MCP_ENABLE_RESOURCES_AS_TOOLS": "true"
      }
    }
  }
}
```

</details>

<details>

<summary>Roo Code</summary>

Use the same `.vscode/mcp.json` configuration as VS Code above.

</details>


### SSL/TLS

If your instance uses a self-signed certificate, either add it to your system's
trusted certificate store or set below for the MCP to allow it to connect:

```bash
export BLACKDUCK_BDSCA_SSL_VERIFY=false
```


## Environment Variables

| Variable                                      | Default             | Description                                                                           |
|-----------------------------------------------|---------------------|---------------------------------------------------------------------------------------|
| `BLACKDUCK_BDSCA_URL`                         | *(required)*        | Base URL of the Black Duck SCA instance                                               |
| `BLACKDUCK_BDSCA_TOKEN`                       | *(none)*            | API bearer token, or use a token file below                                           |
| `BLACKDUCK_BDSCA_TOKEN_FILE`                  | *(none)*            | Path of a file with the token, preferred. Read when `BLACKDUCK_BDSCA_TOKEN` is unset  |
| `BLACKDUCK_BDSCA_SSL_VERIFY`                  | `true`              | Verify SSL/TLS certificates. `false` disables it, or give the path of a CA bundle file. If the file is not found, the default CA bundle is used and a warning is logged |
| `BLACKDUCK_SSL_VERIFY`                        | `true`              | Same as above for all Black Duck products, used when the product-specific variable is unset |
| `BLACKDUCK_BDSCA_LOG_LEVEL`                   | `INFO`              | Logging level: `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`                        |
| `BLACKDUCK_BDSCA_LOG_FORMAT`                  | `colored`           | Log format: `colored`, `json`, `simple`                                               |
| `BLACKDUCK_BDSCA_LOG_FILE`                    | *(none)*            | Path to a log file (written in addition to stderr)                                    |
| `BLACKDUCK_BDSCA_MCP_HOST`                    | `127.0.0.1`         | Address the server listens on with the http transport. The http transport has no authentication, so keep it local |
| `BLACKDUCK_BDSCA_MCP_PORT`                    | `9999`              | Port for the http transport                                                           |
| `BLACKDUCK_BDSCA_MCP_MAX_REQUESTS_PER_SEC`    | `10`                | Limit for inbound requests from the AI assistant, shared by all clients. Bursts of up to twice this are allowed |
| `BLACKDUCK_BDSCA_API_MAX_REQUESTS_PER_SEC`    | `10`                | Limit for outbound API calls to Black Duck SCA                                        |
| `BLACKDUCK_BDSCA_MCP_RESPONSE_MAX_SIZE_BYTES` | `1000000`           | Maximum size in bytes of one tool response or resource read sent to the LLM. Larger text is truncated |
| `BLACKDUCK_MCP_READ_ONLY`                     | `false`             | Turn off the tools that change data or write files, same as `--read-only`             |
| `BLACKDUCK_MCP_DEV_MODE`                      | `false`             | Send unexpected error details to the AI assistant, instead of only the log            |
| `BLACKDUCK_MCP_ENABLE_RESOURCES_AS_TOOLS`     | `false`             | Expose MCP resources as tools, for AI assistants that don't implement the full MCP spec |
| `DETECT_JAVA_PATH`                            | *(none)*            | Path to the `java` executable used for source scanning. Without it, `JAVA_HOME` and then `java` on the `PATH` are used |
| `DETECT_JAR_PATH`                             | *(none)*            | Path to a pre-downloaded Detect JAR. If it does not name an existing file, it is used as the download directory |
| `DETECT_JAR_DOWNLOAD_DIR`                     | `~/detect/download` | Directory where the Detect JAR is downloaded to and looked up from. Takes precedence over `DETECT_JAR_PATH` as the download directory |

The three logging variables can also be set with the CLI flags `--log-level`, `--log-format`
and `--log-file`, which take precedence over the environment variables.

## Authentication

The server reads the API token from the first of these that is set:

1. `BLACKDUCK_BDSCA_TOKEN`, the token itself
2. `BLACKDUCK_BDSCA_TOKEN_FILE`, the path of a file with the token
3. `~/.blackduck/bdsca-token`

A token file keeps the token out of the environment, which most AI assistants pass to every
MCP server and tool they start. Make the file readable by your own user only. The
`health_check` tool shows where the token came from.

API tokens inherit the permissions of their user, so use a dedicated service account with
only the roles you need. See the
[Role and Permission Matrix](https://docs.blackduck.com/r/blackduck/latest/black-duck-documentation/black-duck-user-role-matrix.html).

## Read-only Mode

With `BLACKDUCK_MCP_READ_ONLY=true`, or the `--read-only` flag, these tools are turned off:

`update_bom_component`, `comment_bom_component`, `update_vulnerability_remediation`,
`apply_component_mappings`, `match_component_to_new_origin`, `scan`, `create_report`

In this mode only the tools that are marked read-only stay available, and a turned-off tool is
hidden from the assistant and refused if called. Resources are all reads, so they stay.

## Tools

You can always ask your AI assistant to list the tools it has, e.g. "What Black Duck SCA
tools do you have access to and what are their functions?"

### Dashboard
- `get_dashboard_summary` — Instance-wide dashboard: totals (Projects, Project Versions, Components, Vulnerabilities, scanned code size), this week's security activity, top Components by security risk, and Vulnerability severity breakdown over a configurable period (default: last 24h). A section that fails is returned as null with an error note.

### Projects
- `search_projects_versions` — Search Projects by name or risk filters (security, license, operational, policy violations), or search Versions within a Project.
- `fetch_project_components` — Get the BOM of a Project Version, with filters for severity, vulnerable only and policy violations.
- `fetch_project_vulnerabilities` — Get the Vulnerability summary of a Project Version: counts by severity with a resource URI.
- `update_vulnerability_remediation` — Bulk-update the remediation status of Vulnerabilities (e.g. `IGNORED`, `MITIGATED`, `PATCHED`).

### BOM Components
- `update_bom_component` — Update one or more BOM component entries in a Project Version: review status, usages, licenses, match types and more. One component updates the component or its version; two or more run a bulk update with fewer fields.
- `comment_bom_component` — Post a comment on one or more BOM component entries.

### Match Review
- `find_project_versions_with_match_review_components` — Find the Project Versions with the most match review work: unmatched components and unknown component versions.
- `suggest_unmatched_component_mappings` — Suggest Knowledge Base component, version and origin matches for match review rows.
- `apply_component_mappings` — Apply the user-approved Knowledge Base matches to match review rows.
- `match_component_to_new_origin` — Create a new custom origin on a Knowledge Base component version and match to it.

### LTS Projects (read-only)
- `search_lts_projects_versions` — List LTS Projects, or list LTS Versions within a Project. LTS lives in a separate `/lts-projects/*` API namespace and is not interchangeable with the regular Project tools.
- `fetch_lts_project_components` — Get the BOM of an LTS Project Version, with a severity filter.

### Components
- `search_components` — Search for Components by name across all Projects, list the versions of a Component, or find which Projects use a specific Component version.

### Vulnerabilities
- `search_vulnerabilities` — Search the global Vulnerability database, filtered by severity, source, exploit availability, BDSA tags, score thresholds and more.

### Policies
- `fetch_policy_violation_status` — Get the policy violation status of a Project Version, including violation counts by severity.

### Scanning
- `scan` — One tool for all inputs: BDIO, SBOM (SPDX/CycloneDX), binary and container images. It detects the scan type from the file extension and content, uploads, waits for completion and returns a results summary. Source scanning uses Detect and needs Java.
- `get_scan_status` — Get the status of a scan by its scan ID (`IN_PROGRESS`, `COMPLETE`, `ERROR`, ...), for all scan types.
- `search_scans` — List the code locations of a Project Version, or the scan summaries within a code location.
- `match_code_snippet` — Match code snippets against the Black Duck SCA Knowledge Base to find potential open-source license issues (e.g. GPL/AGPL contamination). Takes snippet text, or a file path with an optional line range.

### Reports
- `create_report` — Generate SBOM, VEX/CSAF or Notices (license) reports for a Project Version. Supports multiple SBOM templates, configurable license categories and including remediation comments.

### Documentation
- `search_documentation` — Search the Black Duck SCA docs at https://docs.blackduck.com for how-to questions. When the instance has AI Assisted Documentation on (2026.1.0 and newer), it searches through the instance and adds its AI answer; otherwise, or with a `version` like `2025.10`, it searches the documentation site directly. Each topic has a `docs-topic://` URI for its full text.

### Status
- `health_check` — Full setup and connectivity check: Python and server version, registered tool and resource counts, relevant environment variables (tokens redacted), Java availability, Detect JAR cache state, and a live connection test against the instance.

## Resources

Resources are read-only data the assistant loads on demand. Most return CSV and support
pagination with `?offset=&limit=` (defaults: offset 0, limit 100).

**BOM and components**
- `components://{project_id}/{project_version_id}` — BOM components. Filters: `?min_severity=CRITICAL|HIGH|MEDIUM|LOW`, `?only_vulnerable=true`, `?only_policy_violations=true`, `?include_upgrade_guidance=true`.
- `component-matched-files://{project_id}/{version_id}/{component_id}{?component_version_id,origin_id,offset,limit}` — Scanned files matched to a BOM component.
- `matched-files://{project_id}/{version_id}{?offset,limit}` — Scanned files matched to any component in a Project Version, with the match type (`FILE_DEPENDENCY_DIRECT`, `FILE_DEPENDENCY_TRANSITIVE`, `BINARY`, `PARTIAL_FILE`, `SNIPPET`).
- `bom-component-comments://{project_id}/{project_version_id}/{component_id}{?component_version_id}` — Comments on a BOM component entry, newest first. Read these before changing a component.
- `kb-components://{query}{?offset,limit}` — Free-text search of the Knowledge Base components by name (CSV, relevance ranked). The `search_components` tool with `action="kb_search"` returns the first page and this URI for the rest. Percent-encode the query.
- `component-versions://{component_id}` — Versions of a Component. Supports `?version_name=` to verify specific versions cheaply and `?sort=releasedon|versionname asc|desc`.
- `component-version-references://{component_id}/{component_version_id}` — Projects using a specific Component version.
- `search-components-in-use://{query}{?offset,limit,min_severity,only_vulnerable}` — Search Components in use across all Project BOMs.
- `upgrade-guidance://{component_id}/{component_version_id}{?origin_id}` — Short-term and long-term upgrade recommendations with Vulnerability risk.

**Vulnerabilities**
- `project-vulnerabilities://{project_id}/{version_id}` — Vulnerabilities of a Project Version. Filters: `?severity=CRITICAL,HIGH,...`, `?component_id=&component_version_id=&origin_id=`, `?triaged=true` (also include Vulnerabilities that already have a remediation status; by default only non-triaged ones are returned).
- `vulnerabilities://{vuln_id}{?include,offset,limit,sort}` — Full details of one CVE, BDSA or EUVD Vulnerability, including CVSS, solution and links. `?include=ranges` appends the affected Component version ranges (BDSA records only).
- `vulnerability-components://{project_id}/{version_id}/{vuln_id}{?related_vuln_id}` — Affected Components and origins for a Vulnerability in a Project Version.
- `vulnerability-affected-projects://{vuln_id}` — Projects and Versions affected by a Vulnerability.
- `vulnerability-related://{vuln_id}` — Every Vulnerability related to a CVE, BDSA or EUVD ID, one row per related record.
- `component-version-vulnerabilities://{component_id}/{component_version_id}{?origin_id,offset,limit}` — Knowledge Base Vulnerabilities for one Component version, highest score first.
- `search-vulnerabilities://` — Vulnerability search results. Filters include `?q=`, `?vulnerabilitySeverity=`, `?exploitAvailable=`, `?workaroundAvailable=`, `?firstDetected=`, `?bdsaTag=` and `?vulnerabilitySource=`.
- `cwes://{cwe_id}` — Name, description and consequences of a CWE weakness, e.g. `cwes://CWE-79`.

**Licenses and policies**
- `licenses://{?name,offset,limit}` — Knowledge Base license catalogue; `name` is a substring match.
- `license-terms://{license_id}{?offset,limit}` — Terms of one license, using a `license_id` from `licenses://`.
- `license-risk-summary://{project_id}/{project_version_id}` — License risk counts without fetching the full BOM.
- `policies://` — All Policies configured in the instance.
- `policy-violation-status://{project_id}/{project_version_id}` — Policy violation status of a Project Version.
- `component-policy-violations://{project_id}/{version_id}/{component_id}/{component_version_id}` — Policy violations for a specific Component.

**LTS**
- `lts-components://{project_id}/{project_version_id}` — LTS BOM components, with `?min_severity=`. Rows carry `vuln_ids` inline and flat Vulnerability counts.
- `lts-project-vulnerabilities://{project_id}/{project_version_id}` — LTS Vulnerabilities, with `?severity=`.
- `lts-vuln-risk-profile://{project_id}/{project_version_id}` — LTS Vulnerability severity counts.

**Users**
- `users://current` — The authenticated user and their role assignments.
- `users://role-descriptions` — Descriptions of all user roles in the instance.
- `user-groups://` — User groups the current user belongs to, with roles.
- `project-groups://` — Project groups the current user has access to, with roles.

**Guidance and documentation**
- `docs-topic://{map_id}/{content_id}` — Full text of one https://docs.blackduck.com topic (markdown).
- `rendering-guidelines://` — Shared formatting guidelines for tool output.
- `workflow-instructions://{name}` — Step-by-step workflow instructions. Names: `vulnerability-investigation`, `finding-vulnerable-projects`, `vulnerability-remediation`, `license-analysis`, `policy-violations`, `license-information`, `scan`, `bom-summary`, `bom-component-update`, `bom-component-comments`, `report-generation`, `match-review`, `dependency-vulnerability-check`.

If your AI assistant cannot read resources, set `BLACKDUCK_MCP_ENABLE_RESOURCES_AS_TOOLS=true`
to expose them as tools.

## First Steps

To get started, ask your assistant some starter questions:

```
Am I connected to the BDSCA instance?
What is the status of the BDSCA instance?
```

These make the assistant call `health_check` and return the MCP server version, the instance
version and the configured environment variables. You can then continue with:

```
Give me an overview of my BDSCA instance
Give me an overview of my security posture
```

The assistant returns the amount of Projects, Components and Vulnerabilities and the top
security risks, and usually suggests which Project Versions to drill into.

### Exploring Projects, Versions, and BOMs

```
Find the Project "my-project" and list its Project Versions
Show me the BOM for my-project 1.4.0
What Vulnerabilities affect my-project 1.4.0?
```

### Cross-project Component Lookup

```
Which Projects use the log4j-core 2.14.1?
Search for Components matching "spring-boot"
```

### Vulnerability Search and Triage

```
Which of my Projects are affected by the CVE-2021-44228?
```

Asking the assistant to triage fetches the Project's Vulnerabilities and then batch-applies a
remediation status like `NOT_AFFECTED`, `MITIGATED` or `PATCHED` with a justification and
comments.

### Policy Compliance

```
What policy violations do I have in my projects?
Which policies does my-project 1.4.0 violate?
```

### Match Review

```
Which of my Project Versions have the most unmatched components?
Suggest Knowledge Base matches for the unmatched components in my-project 1.4.0
```

### Scanning

```
Scan this <directory path> for Components and Vulnerabilities
Scan this <file path> SBOM file / container image / binary
```

### Reports

```
Generate an SBOM in CycloneDX format for my-project 1.4.0
Create a VEX/CSAF document for my-project 1.4.0
Export a Notices report for my-project 1.4.0
```

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

### Verify startup

To check whether the server starts outside your AI assistant, run the `uvx` command from
the installation instructions in a terminal:

```bash
uvx --managed-python --python 3.13 --from 'blackduck-sca-mcp' bdsca
```

Set `BLACKDUCK_BDSCA_URL` and `BLACKDUCK_BDSCA_TOKEN` in the terminal environment first.
The server runs in the foreground; press `Ctrl-C` to stop it after checking the output.

### Check the first log lines

On a successful start the log will contain a line like:

```
INFO  BDSCA MCP server starting  version=1.2.3  bdsca_url=https://...  token=eyJh...9Qw
```

This line is only logged when both the URL and a token were found. If it is missing, look for
a `CRITICAL` line instead:

- `BLACKDUCK_BDSCA_URL is not set. ...` means the URL was not passed to the MCP process.
- `No BDSCA token found. Set BLACKDUCK_BDSCA_TOKEN, or put the token in the file ...` means
  none of `BLACKDUCK_BDSCA_TOKEN`, `BLACKDUCK_BDSCA_TOKEN_FILE` and `~/.blackduck/bdsca-token`
  gave a token.

Check your MCP client configuration and restart. In both cases the server still starts, but
every tool that needs the API fails.

### Run the status check

Ask your AI assistant:

> "Check the Black Duck MCP server status"

This calls `health_check` and returns a full diagnostic snapshot: Python and server
version, registered tool and resource counts, all relevant environment variables
(with tokens redacted), Java availability for source scanning, Detect JAR cache state, and a
live connection test against the Black Duck instance. It is the quickest way to confirm that
everything is configured and reachable before running any other tool.

### Unable to access resources

**Symptom:** MCP calls produce an error like

> "Fetched resource, but the following was an invalid URL: workflow-instructions://vulnerability-remediation"

This is caused when the AI harness is not able to use resources or templated resources
provided by the MCP server. To work around that, expose those resources as tools by setting
`BLACKDUCK_MCP_ENABLE_RESOURCES_AS_TOOLS=true`.

### Token is wrong or expired

**Symptom:** `health_check` returns an error, or every tool call fails with an
authentication error.

1. Open the log file and look for a line containing `Could not authenticate with token`
   or `api_call  status=401`.
2. Verify the token in your Black Duck instance: log in, go to **User → My Access Tokens**
   and confirm the token is active and not expired.
3. Update the token in your MCP client configuration and restart.

> **Note:** API tokens are user-specific. If the token owner's account is disabled, existing
> tokens may be invalidated.

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

### Insufficient permissions

**Symptom:** A scan fails with "Insufficient permissions to run scan."

The message and the log show the missing roles, e.g. `Missing scan access role. Required one of:
GLOBAL_CODE_SCANNER, ...`. Ask a Black Duck administrator to assign one of the required roles
to the account behind your API token. Writes to existing data, such as the remediation status
or BOM component updates, are checked by Black Duck itself, so there the error comes from the
API call. Match review writes check for the BOM Update permission up front and name the roles that
grant it. See the
[Role and Permission Matrix](https://docs.blackduck.com/r/blackduck/latest/black-duck-documentation/black-duck-user-role-matrix.html).

| Operation | Minimum required role (from smallest to largest permissions) |
|-----------|--------------------------------------------------------------|
| Run Scans | `Project Code Scanner` or `Project Group Code Scanner` or `Global Code Scanner` |
| Create new Projects (in addition to the scan role) | `Project Creator` or `Project Group Manager` or `Project Group Administrator` or `Full Access` |
| Create new Project Versions (in addition to the scan role) | `Project Manager` or `Project Administrator` or `Project Group Manager` or `Project Group Administrator` or `Global Project Manager` or `Global Project Administrator` or `Lite Project Manager` or `Lite Global Project Manager` or `Full Access` |

### Java not found (source scanning)

**Symptom:** A source scan fails with a Java-related error, e.g.
`Java executable not found: 'java'`.

Black Duck Detect, used for source, directory and Docker Inspector scans, needs Java 11 or
later. The server looks for Java in this order:

1. `DETECT_JAVA_PATH` (direct path to the `java` executable)
2. `JAVA_HOME`
3. `java` on the system `PATH`

Check that Java is installed with `java -version`. If it is not on the `PATH`, set
`DETECT_JAVA_PATH=/path/to/jdk/bin/java` in your MCP configuration.

### Detect JAR download blocked (source scanning)

**Symptom:** A source scan fails with `Could not obtain Detect JAR` or
`Failed to contact repository`.

The server downloads the Detect JAR from `https://repo.blackduck.com` on first use. If a
firewall blocks that host:

- **Pre-download the JAR** from `repo.blackduck.com` and set
  `DETECT_JAR_PATH=/path/to/detect-X.Y.Z.jar`

### Rate limiting

**Symptom:** Tool calls return "Rate limit exceeded" or slow down under heavy use.

The server applies two independent limits, both `10` requests per second by default:

| Limit | Environment variable |
|-------|----------------------|
| Inbound MCP requests (from the AI assistant) | `BLACKDUCK_BDSCA_MCP_MAX_REQUESTS_PER_SEC` |
| Outbound API calls (to Black Duck SCA) | `BLACKDUCK_BDSCA_API_MAX_REQUESTS_PER_SEC` |

If you have exclusive use of the instance, raise both. If you share it, keep the outbound
limit conservative to avoid affecting other users.
