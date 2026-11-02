# 0.11.0
- Add scan, triage, version override and report tools to the BDBA MCP server, plus resources for components, vendor components, licenses and CVEs, bringing it to feature parity with the BDSCA MCP server
- Add BDBA `rescan_scan` and `download_scan_binary` tools, and give a clear next step when a scan is too old to be read through API v2
- Add a `search_documentation` tool and a `docs-topic://` resource to both MCP servers, to answer how-to questions from the Black Duck documentation
- Add Knowledge Base lookups to the BDSCA MCP server: `search_components` can now search components by name (`kb_search`) and check a specific component version for vulnerabilities by purl or CPE (`kb_version_info`), for example before adding a dependency
- Add `licenses://`, `license-terms://`, `cwes://` and `vulnerability-related://` resources to the BDSCA MCP server, and show CWE IDs, related CVE/BDSA/EUVD records, BDSA patch and vendor-upgrade references, workarounds and affected version ranges on vulnerabilities
- Fix `search_vulnerabilities` ignoring the search term, so CVE ID and free-text searches now return results; an unconstrained search is now rejected with an explanation, and invalid filter values and IDs are reported as errors instead of returning empty results
- Fix several Match Review issues in the BDSCA MCP server: confirming a single component, the scope of the audit comment on version mappings, and the component suggestions
- Explain in the BDSCA `search_vulnerabilities` summary how many rows differ from the requested severity filter, as the BDSCA matches the filter against every CVSS version
- Cap the size of resource responses at 1 MB like tool responses, reject out-of-range pagination limits with an error instead of silently clamping them, and show `total_count` at the top of paginated CSV results
- Treat third-party and scan-derived content as untrusted data in the instructions given to the LLM, and restrict the BDSCA `create_report` output directory to the working directory or a subdirectory
- Fix the BDSCA `scan` tool refusing a project group named `Black Duck Project Groups`, and the empty license family column in `components://`
- Rename `check_status` to `health_check` (breaking change), report the Detect version and discovery mode support in it, and report configuration errors through it instead of failing to start
- Make a failing BDBA `scan_file` say which step failed and whether a product may have been left behind

# 0.9.8
- Update certifi to 2026.10.19
- Update blackduck-sca-api-client to 0.3.8, which updates protobuf to 7.36.2 and urllib3 to 2.8.0
- Remove the BDBA MCP server from the package and the plugin marketplace in this patch release
- Fix the plugin `uvx` arguments, which gave `--python 3.13` as one argument and made uvx fail

# 0.9.7
- Add support for the Component Match Review feature
- Add `component-matched-files://` and `matched-files://` resources to look up which scanned project files matched a component
- Add the `update_bom_component` tool to update the metadata of BOM components and component versions, one or many at a time
- Add the `comment_bom_component` tool and the `bom-component-comments://` / `bom-component-version-comments://` resources

# 0.9.6
- Fix installation pulling in an incompatible FastMCP 4.x release by pinning it below 4.0.0

# 0.9.5
- Report the LLM client (harness) in the `check_status` tool, to help with troubleshooting
- Fix `check_status` reporting incorrect resource and tool counts

# 0.9.4
- Remove the Unified MCP server; use the separate BDSCA and BDBA servers
- Add `?include_upgrade_guidance=true` to the `components://` resource for short-term / long-term upgrade columns
- Prefer the Origin-scoped upgrade guidance when BOM rows have Origin IDs
- Add read-only support for LTS (Long-Term Support) Projects and Project Versions

# 0.9.3
- Add more logging for troubleshooting
- Allow Detect scans to take custom Detect parameters
- Use the stdio transport by default, which simplifies the MCP server configuration
- Fix log level parameter case sensitivity
- Replace the common `SSL_VERIFY` env var with product-specific ones (breaking change)

# 0.9.2
- Rename the logging env vars to use the product namespace (breaking change)

# 0.9.1
- Fix resource loading in mcp

# 0.9.0
- Fix tools failing to load when the MCP server is installed as a package, by removing dynamic tool loading

# 0.8.2
- Include the LLM instruction files in the package
- Remove the BDBA MCP server from the Claude plugin config

# 0.8.1
- Rename the mcp package from `rubberduck` to `blackduck-sca-mcp`
- Update the `blackduck-sca-api-client` dependency to pick up fixes

# 0.8.0
- Remove unused prompts
- Remove the dump session tool; use your client's export function instead
- Redact user information from all MCP responses to avoid leaking personal data to the LLM

# 0.7.0
- Add support for running Detect scans

# 0.6.3
- Improve `update_vulnerability_remediation` to return `submitted`, the BDSCA response and a `verification_uri_template` for reading back the remediation state
- Use query parameters instead of path segments for `offset` and `limit` in paginated resource URIs, defaulting to 0 and 100
- Add configurable rate limiting of requests from the LLM and of requests to BDSCA and BDBA
- Detect the BDSCA version and warn when it is older than 2025.10.0, which some features need
- Add configurable SSL/TLS verification
- Add a User-Agent header for BDSCA and BDBA requests

# 0.6.2
- Fix scanning for non-sysadmin users without access to the root project group
- Add Policy Violation status support
- Make the LLM instructions more reliably followed by using RFC 2119 language (MUST, SHOULD, etc.)

# 0.6.1
- Fix the BDBA MCP server failing to start when installed as a package

# 0.6.0
- Add `--log-level`, `--log-format` and `--log-file` CLI options to all MCP servers
- Load the workflow instructions on demand as MCP Resources
- Add the `search_scans` MCP Tool for listing Code Locations and Scan summaries
- Add structured `renderingInstructions` to all MCP tool responses to guide how the LLM formats output
- Add YAML SBOM format support and fix SBOM file extensions for CycloneDX vs. SPDX

# 0.5.3
- Fix the MCP servers failing to start when installed as a package

# 0.5.2
- Add a Claude plugin with skills and agents for vulnerability triage, project scanning and compliance reporting

# 0.5.0
- Initial release
