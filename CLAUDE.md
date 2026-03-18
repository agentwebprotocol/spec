# AWP Specification

This repo contains the authoritative Agent Web Protocol specification (SPEC.md).

## Ecosystem

| Repo | Role |
|------|------|
| **spec** (this repo) | Source of truth — the specification |
| agentwebprotocol.org | Standards website — fetches SPEC.md live from this repo |
| agent-json.org | Schema reference and validator site |
| agent.json | npm CLI (`npx agent-json init`) |
| mcp-server | MCP server for Claude Code — has spec embedded |

GitHub org: github.com/agentwebprotocol

## When You Change SPEC.md

- agentwebprotocol.org/spec picks up changes within 1 hour (live fetch with revalidate: 3600)
- agentwebprotocol.org/llms-full.txt has an embedded copy — must be manually updated
- The MCP server (awp-mcp-server npm package) has an embedded copy — must bump version and `npm publish`
- The agent-json.org validator/schema may need updating if fields change

## Key Files

- `SPEC.md` — the specification (366 lines, 16 sections)
- `README.md` — project overview
- `LICENSE` — MIT

## Spec Version

Current: v0.1 (Draft RFC, published 2026-03-16)

## Contribution

Changes proposed via GitHub issues and pull requests. Contact: spec@agentwebprotocol.org
