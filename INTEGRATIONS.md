# Integrations

External software this project talks to. No secret values here — credentials are declared (by name only) in Varlock's `.env.schema` or the platform's credential store.

| System | Purpose | How |
| --- | --- | --- |
| Brave Search | Web search | REST API key |
| GitHub | Code/repo search | REST/Search API token |
| News / X / Reddit / YouTube / Discord and more | Optional engines when keyed | per-provider API keys, named multi-accounts |
| OmniRoute | Gateway-only credentials | see docs/accounts-and-gateways.md |
| MCP clients (Claude Desktop, Cursor, …) | Consumers | rmcp stdio / streamable HTTP |

Provider list and auth details: [README.md](README.md) and [docs/accounts-and-gateways.md](docs/accounts-and-gateways.md).
