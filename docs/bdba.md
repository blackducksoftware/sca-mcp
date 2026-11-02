# Black Duck Binary Analysis (BDBA) MCP Server

The BDBA MCP server connects your AI assistant to a Black Duck Binary Analysis (BDBA)
instance. It gives access to BDBA groups (projects), products (scans) and the component
analysis results of binary file scans, and can scan files, triage Vulnerabilities, override
component versions and generate reports.

For an overview of both servers and the shared security guidance, see the
[main README](../README.md). For the Black Duck SCA server, see [BDSCA](bdsca.md).

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

- **Black Duck Binary Analysis instance** with a user account and API token
- **Python 3.13** and **[uv](https://docs.astral.sh/uv/getting-started/installation/)**

## Installation

The server is started with the `bdba` command from the `blackduck-sca-mcp` package. Add it
to your AI assistant as described below. The full list of settings is in
[Environment Variables](#environment-variables).

<details>

<summary>Claude Code</summary>

```bash
claude mcp add blackduck-bdba-mcp \
  --env BLACKDUCK_BDBA_URL=https://<your-instance-url> \
  --env BLACKDUCK_BDBA_TOKEN=<api-token> \
  -- uvx \
  --managed-python --python 3.13 \
  --from 'blackduck-sca-mcp' bdba
```

</details>

<details>

<summary>Claude Desktop (Cowork)</summary>

Add via UI and make sure the config file has similar values afterward.

```json
{
    "managedMcpServers": [
    {
      "name": "blackduck-bdba-mcp",
      "source": "user",
      "command": "/bin/uvx",
      "args": [
        "--managed-python",
        "--python", "3.13",
        "--from", "blackduck-sca-mcp", "bdba"
      ],
      "env": {
        "BLACKDUCK_BDBA_URL": "https://<your-instance-url>",
        "BLACKDUCK_BDBA_TOKEN": "<api-token>"
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
    "blackduck-bdba-mcp": {
      "command": "uvx",
      "args": [
        "--managed-python",
        "--python", "3.13",
        "--from", "blackduck-sca-mcp", "bdba"
      ],
      "env": {
        "BLACKDUCK_BDBA_URL": "https://<your-instance-url>",
        "BLACKDUCK_BDBA_TOKEN": "<api-token>"
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
copilot mcp add blackduck-bdba-mcp \
  --env BLACKDUCK_BDBA_URL=https://<your-instance-url> \
  --env BLACKDUCK_BDBA_TOKEN=<api-token> \
  --env BLACKDUCK_MCP_ENABLE_RESOURCES_AS_TOOLS=true \
  -- uvx \
  --managed-python --python 3.13 \
  --from 'blackduck-sca-mcp' bdba
```

```json
{
  "servers": {
    "blackduck-bdba-mcp": {
      "command": "uvx",
      "args": [
        "--managed-python",
        "--python", "3.13",
        "--from", "blackduck-sca-mcp", "bdba"
      ],
      "env": {
        "BLACKDUCK_BDBA_URL": "https://<your-instance-url>",
        "BLACKDUCK_BDBA_TOKEN": "<api-token>",
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
export BLACKDUCK_BDBA_SSL_VERIFY=false
```


## Environment Variables

| Variable                                       | Default             | Description                                                                           |
|------------------------------------------------|---------------------|---------------------------------------------------------------------------------------|
| `BLACKDUCK_BDBA_URL`                           | *(required)*        | Base URL of the Black Duck BDBA instance                                              |
| `BLACKDUCK_BDBA_TOKEN`                         | *(none)*            | API bearer token, or use a token file below                                           |
| `BLACKDUCK_BDBA_TOKEN_FILE`                    | *(none)*            | Path of a file with the token, preferred. Read when `BLACKDUCK_BDBA_TOKEN` is unset   |
| `BLACKDUCK_BDBA_SSL_VERIFY`                    | `true`              | Verify SSL/TLS certificates. `false` disables it, or give the path of a CA bundle file. If the file is not found, the default CA bundle is used and a warning is logged |
| `BLACKDUCK_SSL_VERIFY`                         | `true`              | Same as above for all Black Duck products, used when the product-specific variable is unset |
| `BLACKDUCK_BDBA_PREFERRED_VULNERABILITY_DATABASE` | `BDSA`           | Whose identifier represents a group of related Vulnerability matches, e.g. a CVE match and its corresponding BDSA advisory match |
| `BLACKDUCK_BDBA_LOG_LEVEL`                     | `INFO`              | Logging level: `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`                        |
| `BLACKDUCK_BDBA_LOG_FORMAT`                    | `colored`           | Log format: `colored`, `json`, `simple`                                               |
| `BLACKDUCK_BDBA_LOG_FILE`                      | *(none)*            | Path to a log file (written in addition to stderr)                                    |
| `BLACKDUCK_BDBA_MCP_HOST`                      | `127.0.0.1`         | Address the server listens on with the http transport. The http transport has no authentication, so keep it local |
| `BLACKDUCK_BDBA_MCP_PORT`                      | `9998`              | Port for the http transport                                                           |
| `BLACKDUCK_BDBA_MCP_MAX_REQUESTS_PER_SEC`      | `10`                | Limit for inbound requests from the AI assistant, shared by all clients. Bursts of up to twice this are allowed |
| `BLACKDUCK_BDBA_API_MAX_REQUESTS_PER_SEC`      | `10`                | Limit for outbound API calls to BDBA                                                  |
| `BLACKDUCK_BDBA_MCP_RESPONSE_MAX_SIZE_BYTES`   | `1000000`           | Maximum size in bytes of one tool response or resource read sent to the LLM. Larger text is truncated |
| `BLACKDUCK_MCP_READ_ONLY`                      | `false`             | Turn off the tools that change data or write files, same as `--read-only`             |
| `BLACKDUCK_MCP_DEV_MODE`                       | `false`             | Send unexpected error details to the AI assistant, instead of only the log            |
| `BLACKDUCK_MCP_ENABLE_RESOURCES_AS_TOOLS`      | `false`             | Expose MCP resources as tools, for AI assistants that don't implement the full MCP spec |

The three logging variables can also be set with the CLI flags `--log-level`, `--log-format`
and `--log-file`, which take precedence over the environment variables.

## Authentication

The server reads the API token from the first of these that is set:

1. `BLACKDUCK_BDBA_TOKEN`, the token itself
2. `BLACKDUCK_BDBA_TOKEN_FILE`, the path of a file with the token
3. `~/.blackduck/bdba-token`

A token file keeps the token out of the environment, which most AI assistants pass to every
MCP server and tool they start. Make the file readable by your own user only. The
`health_check` tool shows where the token came from.

API tokens inherit the permissions of their user, so use a dedicated account with only the
access you need.

## Read-only Mode

With `BLACKDUCK_MCP_READ_ONLY=true`, or the `--read-only` flag, these tools are turned off:

`scan_file`, `rescan_scan`, `download_scan_binary`, `create_triage`, `update_triage`,
`delete_triage`, `create_version_override`, `delete_version_override`, `create_report`,
`get_report`

In this mode only the tools that are marked read-only stay available, and a turned-off tool is
hidden from the assistant and refused if called. Resources are all reads, so they stay.

## Tools

You can always ask your AI assistant to list the tools it has, e.g. "What Black Duck BDBA
tools do you have access to and what are their functions?"

### Groups & Scans
- `search_groups` — Search BDBA groups (projects) by name.
- `search_scans` — Search BDBA products (scans) by name, within a group or globally across all groups.
- `scan_file` — Scan a local file or a remote reference with BDBA, optionally waiting for the results.
- `download_scan_binary` — Download the binary BDBA stored for a scan to a local file.
- `rescan_scan` — Rescan an existing scan in place. Works if BDBA still has the scanned file or knows where to fetch it from.

### Components Catalog
- `search_components` — Search the global BDBA supported-component catalog by name.

### Reports
- `create_report` — Generate a report (CycloneDX, SPDX, CSV, VEX/triages, BDIO) for a scan.
- `get_report` — Check a report's status and, once ready, download its content.

### Triages
- `search_triages` — Search Vulnerability triages (suppressions and remediations).
- `create_triage` — Create a triage for a component and Vulnerability combination.
- `update_triage` — Partially update an existing triage.
- `delete_triage` — Delete a triage.

### Version Overrides
- `search_version_overrides` — Search version overrides.
- `create_version_override` — Make BDBA treat matched components as a specific version.
- `delete_version_override` — Delete a version override.

### Documentation
- `search_documentation` — Search the BDBA docs at https://docs.blackduck.com for how-to questions. Each topic has a `docs-topic://` URI for its full text.

### Status
- `health_check` — Full setup and connectivity check: Python and server version, registered tool and resource counts, relevant environment variables (tokens redacted), and a live connection test against the BDBA instance.

## Resources

Resources are read-only data the assistant loads on demand. Most return CSV and support
pagination with `?offset=&limit=` (defaults: offset 0, limit 100).

**Scan results**
- `components://{product_id}` — BOM components of a scan: `component_name`, `version`, `codetype`, `result_component_id`, `is_vendor_component`, `component_id`, `vuln_count` and `vuln_ids` (semicolon-separated). When `is_vendor_component` is `true`, `component_id` is a vendor component ID: use `vendor-component://`, not the catalog, and don't research the component further.
- `result-component://{component_id}` — File-level match evidence for one BOM component.

**Component catalogs**
- `components-catalog://` — The global supported-component catalog.
- `component://{component_id}` — Full detail for one catalog component (JSON).
- `vendor-components://` — Vendor components, the organization's own taught components.
- `vendor-component://{vendor_component_id}` — Full detail for one vendor component (JSON).
- `licenses://` — License list.

**Triages and overrides**
- `triages://` — Vulnerability triages.
- `triage://{triage_id}` — Full detail for one triage (JSON).
- `version-overrides://` — Version overrides.
- `version-override://{version_override_id}` — Full detail for one version override (JSON).

**Vulnerabilities**
- `cves://` — The BDBA Vulnerability knowledge base (CVEs and BDSAs).
- `vulnerability://{vulnerability_id}` — Full detail for one CVE or BDSA (JSON).

**Guidance and documentation**
- `docs-topic://{map_id}/{content_id}` — Full text of one https://docs.blackduck.com topic (markdown).
- `rendering-guidelines://` — Shared formatting guidelines for tool output.
- `workflow-instructions://{name}` — Step-by-step workflow instructions. Names: `scan`, `component-detail`, `report-generation`, `triage`, `version-override`.

If your AI assistant cannot read resources, set `BLACKDUCK_MCP_ENABLE_RESOURCES_AS_TOOLS=true`
to expose them as tools.

## First Steps

To get started, ask your assistant some starter questions:

```
Am I connected to the BDBA instance?
What is the status of the BDBA instance?
```

These make the assistant call `health_check` and return the MCP server version, the instance
connection status and the configured environment variables.

### Finding Groups and Scans

```
Find the BDBA group "firmware" and list its scans
Search the scans for "router-image"
```

### Scanning a Binary

```
Scan <file path> with BDBA and show me the components and Vulnerabilities
Rescan the scan "router-image 2.1"
```

### Reviewing Components

```
Show me the components found in the scan "router-image 2.1"
Which of those components are vulnerable?
Show me the file-level match evidence for openssl in that scan
```

### Triage and Version Overrides

```
Triage CVE-2021-44228 in that scan as not affected, with a justification
BDBA matched zlib as 1.2.8 but it is 1.2.11. Create a version override
```

### Reports

```
Generate a CycloneDX SBOM for the scan "router-image 2.1"
Create a VEX report for the same scan
```

## Troubleshooting

### Enable debug logging to a file

The fastest way to diagnose any issue is to capture a full debug log. Add these two
environment variables to your MCP server configuration:

```
BLACKDUCK_BDBA_LOG_LEVEL=DEBUG
BLACKDUCK_BDBA_LOG_FILE=/tmp/bdba-mcp.log
```

**Claude Code example** — re-add the server with logging enabled:

```bash
export BLACKDUCK_BDBA_LOG_LEVEL=DEBUG
export BLACKDUCK_BDBA_LOG_FILE=/tmp/bdba-mcp.log
claude
```

**VS Code / Claude Desktop** — add to the `env` block in your `mcp.json` / config file:

```json
"env": {
  "BLACKDUCK_BDBA_URL": "https://<your-instance-url>",
  "BLACKDUCK_BDBA_TOKEN": "<api-token>",
  "BLACKDUCK_BDBA_LOG_LEVEL": "DEBUG",
  "BLACKDUCK_BDBA_LOG_FILE": "/tmp/bdba-mcp.log"
}
```

Then run the failing operation and inspect `/tmp/bdba-mcp.log`.

### Verify startup

To check whether the server starts outside your AI assistant, run the `uvx` command from
the installation instructions in a terminal:

```bash
uvx --managed-python --python 3.13 --from 'blackduck-sca-mcp' bdba
```

Set `BLACKDUCK_BDBA_URL` and `BLACKDUCK_BDBA_TOKEN` in the terminal environment first.
The server runs in the foreground; press `Ctrl-C` to stop it after checking the output.

### Check the first log lines

On a successful start the log will contain a line like:

```
INFO  BDBA MCP server starting  version=1.2.3  bdba_url=https://...  token=eyJh...9Qw
```

This line is only logged when both the URL and a token were found. If it is missing, look for
a `CRITICAL` line instead:

- `BLACKDUCK_BDBA_URL is not set. ...` means the URL was not passed to the MCP process.
- `No BDBA token found. Set BLACKDUCK_BDBA_TOKEN, or put the token in the file ...` means
  none of `BLACKDUCK_BDBA_TOKEN`, `BLACKDUCK_BDBA_TOKEN_FILE` and `~/.blackduck/bdba-token`
  gave a token.

Check your MCP client configuration and restart. In both cases the server still starts, but
every tool that needs the API fails.

### Run the status check

Ask your AI assistant:

> "Check the Black Duck BDBA MCP server status"

This calls `health_check` and returns a full diagnostic snapshot: Python and server
version, registered tool and resource counts, all relevant environment variables
(with tokens redacted), and a live connection test against the BDBA instance. It is the
quickest way to confirm that everything is configured and reachable before running any other
tool.

### Unable to access resources

**Symptom:** MCP calls produce an error like

> "Fetched resource, but the following was an invalid URL: workflow-instructions://scan"

This is caused when the AI harness is not able to use resources or templated resources
provided by the MCP server. To work around that, expose those resources as tools by setting
`BLACKDUCK_MCP_ENABLE_RESOURCES_AS_TOOLS=true`.

### Token is wrong or expired

**Symptom:** `health_check` returns an error, or every tool call fails with an
authentication error.

1. Check the log file for authentication errors.
2. Verify the token in your BDBA instance: log in and confirm the API token is still valid.
3. Update the token in your MCP client configuration and restart.

> **Note:** API tokens are user-specific. If the token owner's account is disabled, existing
> tokens may be invalidated.

### SSL / self-signed certificate

**Symptom:** Connection fails with a certificate verification error.

If your BDBA instance uses a self-signed or internally-issued certificate:

```
BLACKDUCK_BDBA_SSL_VERIFY=false
```

Or point to your CA bundle:

```
BLACKDUCK_BDBA_SSL_VERIFY=/path/to/ca-bundle.crt
```

### Write operations are missing

**Symptom:** The assistant says it has no tool to scan, triage or create reports.

Check whether `BLACKDUCK_MCP_READ_ONLY` is `true` or the server runs with `--read-only`. See
[Read-only Mode](#read-only-mode) for the tools it turns off.

### Rate limiting

**Symptom:** Tool calls return "Rate limit exceeded" or slow down under heavy use.

The server applies two independent limits, both `10` requests per second by default:

| Limit | Environment variable |
|-------|----------------------|
| Inbound MCP requests (from the AI assistant) | `BLACKDUCK_BDBA_MCP_MAX_REQUESTS_PER_SEC` |
| Outbound API calls (to BDBA) | `BLACKDUCK_BDBA_API_MAX_REQUESTS_PER_SEC` |

If you have exclusive use of the instance, raise both. If you share it, keep the outbound
limit conservative to avoid affecting other users.

### Truncated responses

**Symptom:** A very large BOM or result is cut off.

Responses are limited to `BLACKDUCK_BDBA_MCP_RESPONSE_MAX_SIZE_BYTES` (1 MB by default). For
whole BOMs, ask the assistant to read the `components://{product_id}` resource page by page with
`?offset=&limit=`. You can also raise the limit, at the cost of more data going to the LLM.
