# GitHub multi-client connector research pack

Generated 2026-10-11. This is a research and implementation guide for adapting GitHub's official MCP server across AI clients. It does not create a replacement MCP server or claim lossless conversion between MCP, skills, and agents.

## Source retrieval

Firecrawl successfully scraped three official pages on 2026-10-11:

- [Agent Skills specification](https://agentskills.io/specification)
- [GitHub Copilot custom agents configuration](https://docs.github.com/en/copilot/reference/custom-agents-configuration)
- [Claude Code custom subagents](https://code.claude.com/docs/en/sub-agents)

Additional GitHub MCP and client-configuration details come from the official repositories and documentation linked below. Firecrawl returned cached page content for the three scraped pages; the other sources were retrieved through GitHub tools and direct documentation fetches.

## What the connector is

The official all-toolsets endpoint is:

```text
https://api.githubcopilot.com/mcp/x/all
```

The read-only endpoint is:

```text
https://api.githubcopilot.com/mcp/x/all/readonly
```

Source: [GitHub remote MCP server documentation](https://github.com/github/github-mcp-server/blob/main/docs/remote-server.md).

These endpoints expose GitHub's hosted server. The client must discover the actual tool schemas after authentication. A pasted tool-name list is not a schema.

## Three layers

| Layer | What it supplies | Official schema/source | Conversion boundary |
|---|---|---|---|
| MCP | Live tools and data access | [MCP schema reference](https://modelcontextprotocol.io/specification/latest/schema) and [GitHub MCP server](https://github.com/github/github-mcp-server) | Convert client connection settings, not instructions into executable tools. |
| Skills | Reusable instructions and resources | [Agent Skills specification](https://agentskills.io/specification) | Preserve the skill folder; vendor extensions are not universally portable. |
| Agents | Native worker definitions | [Claude subagents](https://code.claude.com/docs/en/sub-agents), [Copilot custom agents](https://docs.github.com/en/copilot/reference/custom-agents-configuration) | Translate only documented fields; models, permissions, and tool namespaces differ. |

## Firecrawl-enriched schema notes

### Agent Skills

The official specification requires a directory containing `SKILL.md`. Required frontmatter is `name` and `description`. The name must match its parent directory and uses lowercase letters, numbers, and hyphens. Optional fields include `license`, `compatibility`, `metadata`, and experimental `allowed-tools`. Optional directories include `scripts/`, `references/`, and `assets/`. [Source](https://agentskills.io/specification)

The specification recommends keeping `SKILL.md` under 500 lines and validating with `skills-ref validate ./my-skill`. [Source](https://agentskills.io/specification)

### Claude subagents

Claude subagent files live under `.claude/agents/` or `~/.claude/agents/` and require `name` and `description`. Documented fields include `tools`, `disallowedTools`, `model`, `permissionMode`, `skills`, `mcpServers`, `hooks`, and `isolation`. MCP tools use names such as `mcp__github`. [Source](https://code.claude.com/docs/en/sub-agents)

### Copilot custom agents

Copilot agent profiles require `description`. Documented fields include `name`, `target`, `tools`, `model`, `mcp-servers`, and `metadata`. MCP tools use `server/tool` namespacing, such as `github/get_file_contents`. The Markdown prompt can be at most 30,000 characters. [Source](https://docs.github.com/en/copilot/reference/custom-agents-configuration)

## Existing converter projects

| Project | Relevant capability | Source |
|---|---|---|
| `x0c/agentsync` | Shares global instructions, complete skill folders, and MCP configuration across installed clients. | [Repository](https://github.com/x0c/agentsync) |
| `Ditto190/agentsync-multiagent` | Contains client-specific MCP renderers for Claude, Cursor, Gemini, Codex, Grok, Goose, and others. | [Renderer](https://github.com/Ditto190/agentsync-multiagent/blob/main/internal/agentsync/mcp_render.go) |
| `dyoshikawa/rulesync` | Documents import, generation, and conversion of rules, MCP, skills, and subagents. | [Repository](https://github.com/dyoshikawa/rulesync) |

`AGENTS.md` synchronization is not the same as native agent-definition conversion. Rulesync should be tested in a disposable copy before applying it to a working project.

## Included starter files

The ZIP contains:

- Canonical connector manifest.
- Portable `github-research` skill.
- Claude and Copilot research-agent profiles.
- Staged client MCP configurations.
- Renderer and syntax-validation scripts.
- Source index.

The renderer was executed and its generated JSON/TOML syntax was checked. Live authentication, client loading, tool execution, and third-party conversion were not tested.

## Security

Use the read-only endpoint for research. Do not commit tokens. GitHub states that Copilot cloud MCP tools can run autonomously without approval, so instructions alone are not a sufficient write boundary there. [Source](https://docs.github.com/copilot/how-tos/copilot-on-github/customize-copilot/configure-mcp-servers)
