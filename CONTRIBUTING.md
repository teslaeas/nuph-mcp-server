# Contributing

Thanks for your interest in contributing to the nuph.ai MCP Server documentation!

## What lives here

This repository contains:

- 📘 Documentation (setup guides, protocol reference, tool specs)
- ⚙️ Example configurations for various MCP clients
- 🧪 Tested prompt examples by use case
- 🛠️ Troubleshooting guides

**What does NOT live here:**

- The MCP server source code (it's proprietary and hosted at `api.nuph.ai`)
- The nuph.ai platform code

## How to contribute

### 🐛 Report a bug

Found an issue with the MCP server? [Open an issue](https://github.com/nuph-ai/nuph-mcp-server/issues/new).

Include:
- Your MCP client (Claude Desktop, Cursor, etc.) and version
- The exact error message or unexpected behavior
- Steps to reproduce
- What you expected vs. what happened

### 📝 Improve the docs

Typos, clarifications, additional examples, better setup instructions — all welcome!

1. Fork this repo
2. Create a branch (`git checkout -b docs/improve-setup`)
3. Make your changes
4. Open a PR with a clear description

### 💡 Request a feature

Want a new tool exposed through the MCP? Or a better prompt example?

[Start a discussion](https://github.com/nuph-ai/nuph-mcp-server/discussions) — we read all of them.

Before requesting, check:
- Is the action safe for an AI to do on the user's behalf? (e.g. we don't expose `approve_prospect` because users should review in the dashboard)
- Does it fit the MCP model? (stateless, read-leaning, scoped by API key)

### 🌐 Translate

The docs are currently in English. If you want to translate to Spanish, Portuguese, or another language, open an issue to coordinate.

## Code of Conduct

Be kind. Be constructive. Don't be rude to maintainers or other contributors.

## Security issues

Do **not** open a public issue for security vulnerabilities. Email [security@nuph.ai](mailto:security@nuph.ai) instead. See [SECURITY.md](SECURITY.md) for details.

## License

By contributing, you agree that your contributions will be licensed under the MIT License. See [LICENSE](LICENSE).

---

Thanks for making nuph.ai + MCP better 🙌
