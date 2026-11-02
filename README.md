# Black Duck SCA — MCP Servers

[![PyPI - Version](https://img.shields.io/pypi/v/blackduck-sca-mcp)](https://pypi.org/project/blackduck-sca-mcp/)
[![PyPI - License](https://img.shields.io/pypi/l/blackduck-sca-mcp)](https://github.com/blackducksoftware/sca-mcp/blob/main/LICENSE)
[![Website](https://img.shields.io/badge/website-https%3A%2F%2Fwww.blackduck.com-blue)](https://www.blackduck.com/)
[![Email](https://img.shields.io/badge/email-support%40blackduck.com-green)](mailto:support@blackduck.com)


Connect your AI coding assistant to the Black Duck Software Composition Analysis (SCA) products.
The MCP servers expose your Black Duck instances as a set of tools that any
MCP-compatible AI harness can call to perform actions such as:
investigate your software's security posture and vulnerabilities, triage findings,
scan code and binaries, and generate vulnerability reports and SBOMs.

## Table of Contents

1. [Products](#products)
2. [Latest changes](#latest-changes)
3. [Getting Started](#getting-started)
   - [Prerequisites](#prerequisites)
   - [Installation](#installation)
4. [Security](#security)
5. [Support](#support)

## Products

This package ships two MCP servers, one per product. Install the one you have an instance
for, or both. Each has its own guide with the installation steps, the
full list of tools/resources and environment variables, and troubleshooting.

| Product | MCP Guide | Use it to |
|---------|-----------|---------|
| [**Black Duck SCA**](https://www.blackduck.com/software-composition-analysis-tools/black-duck-sca.html) | [BDSCA](https://github.com/blackducksoftware/sca-mcp/blob/main/docs/bdsca.md) | Explore Projects and BOMs, triage Vulnerabilities, check Policy compliance, review component matches, run scans, generate SBOM, VEX/CSAF and Notices reports | 
| [**Black Duck Binary Analysis**](https://www.blackduck.com/software-composition-analysis-tools/binary-analysis.html) | [BDBA](https://github.com/blackducksoftware/sca-mcp/blob/main/docs/bdba.md) | Scan binaries and firmware, review the components found in them, triage Vulnerabilities, override component versions, generate reports | 

You can always check the full list of available tools by asking your AI assistant to list them, e.g.
"What Black Duck SCA tools do you have access to and what are their functions?"

## Latest changes

See the [changelog](https://github.com/blackducksoftware/sca-mcp/blob/main/CHANGELOG.md)
for the latest changes.

## Getting Started

### Prerequisites

- **A Black Duck instance** (SCA, Binary Analysis or both) with a user account and API token
- **Python 3.13**
- **[uv](https://docs.astral.sh/uv/getting-started/installation/)** — Python package manager
- **Java 11+** _(optional)_ — required only for BDSCA source scanning via Detect

Both servers run from the same `blackduck-sca-mcp` package, with `uvx`. 

### Installation

The guides have the setup instructions for various assistants, e.g. Claude Code, Claude Desktop, VS Code, Copilot CLI and
Roo Code:

- [Installing the BDSCA server](https://github.com/blackducksoftware/sca-mcp/blob/main/docs/bdsca.md#installation)
- [Installing the BDBA server](https://github.com/blackducksoftware/sca-mcp/blob/main/docs/bdba.md#installation)

Once connected, ask your assistant "Am I connected to the Black Duck instance?" to check the
setup. If something goes wrong, see the troubleshooting sections of the guides:
- [Troubleshooting BDSCA](https://github.com/blackducksoftware/sca-mcp/blob/main/docs/bdsca.md#troubleshooting)
- [Troubleshooting BDBA](https://github.com/blackducksoftware/sca-mcp/blob/main/docs/bdba.md#troubleshooting)

## Security

### Token permissions

API tokens inherit the full permissions of the associated user account.
If you provide a token with write access, the AI assistant can modify
data in your Black Duck instance — including updating vulnerability
remediation status, policy overrides and triages.

We recommend creating a dedicated service account with the minimum
permissions required for your use case. See the [Role and Permission Matrix](https://docs.blackduck.com/r/blackduck/latest/black-duck-documentation/black-duck-user-role-matrix.html) for details.

### Read-only mode

Set `BLACKDUCK_MCP_READ_ONLY=true`, or start a server with `--read-only`, to turn off the
tools that change data or write files. The tools it turns off are listed in
[Read-only Mode for BDSCA](https://github.com/blackducksoftware/sca-mcp/blob/main/docs/bdsca.md#read-only-mode) and
[for BDBA](https://github.com/blackducksoftware/sca-mcp/blob/main/docs/bdba.md#read-only-mode).

### Token storage

Most AI assistants pass their environment to every MCP server and tool they start, so a token
in an env var can be read by all of them. Each server looks for its token in this order:

1. `BLACKDUCK_BDSCA_TOKEN` or `BLACKDUCK_BDBA_TOKEN`, the token itself
2. `BLACKDUCK_BDSCA_TOKEN_FILE` or `BLACKDUCK_BDBA_TOKEN_FILE`, the path of a file with the token
3. `~/.blackduck/bdsca-token` or `~/.blackduck/bdba-token`, read when neither of the two above
   is set

To keep tokens out of the environment, save each one in a file that only your own user can read:

```bash
mkdir -p ~/.blackduck && chmod 700 ~/.blackduck
printf '%s' "<bdsca-api-token>" > ~/.blackduck/bdsca-token
printf '%s' "<bdba-api-token>" > ~/.blackduck/bdba-token
chmod 600 ~/.blackduck/bdsca-token ~/.blackduck/bdba-token
```

and configure only the URLs. The `health_check` tool shows where the token came from.

### Data sent to your LLM provider

MCP tools results go to the LLM that your AI assistant uses. These may hold
Vulnerability findings, Project, Version and Component names, Policy settings, and Scan
results. The API tokens never go to the LLM.

Black Duck does not control how your LLM provider stores or uses this data, or whether the
provider trains models on it. Before you use the servers with sensitive or regulated
findings, read your provider's terms and data processing agreement, and use its zero data
retention or enterprise options where you need them.

### Error details

Unexpected error details, which can hold URLs and file paths, stay in the server log and do
not go to the AI assistant. Set `BLACKDUCK_MCP_DEV_MODE=true` to have errors shown to the LLM too, 
for example if you are debugging a problem.

## Support

- [Black Duck Website](https://www.blackduck.com/)
- [Community Portal](https://community.blackduck.com)
- [Email](mailto:support@blackduck.com)
