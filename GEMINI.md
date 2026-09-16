# Skarn

This extension adds a local `skarn` MCP server and the `skarn-audit` skill. The server runs on the user's machine, makes no network call, and exposes four tools:

- `scan_sessions` returns findings from past AI coding sessions. A detected credential value appears only in masked form; file paths, session ids, timestamps, and a Compliance API export's user and workspace ids come back unmasked.
- `vet_configs` returns findings on the local assistant configuration. They quote excerpts of hook commands and MCP server definitions, with detected credential values masked.
- `list_sessions` returns session metadata: `session_id`, `cli`, `user_id`, the first and last timestamps, and message, call, and token counts. Never message content.
- `session_stats` returns aggregate counts. Never message content.

Every tool is read-only. None of them writes a file, changes a configuration, or reaches the network.

A masked value is masked because it is a credential. Never reconstruct one, never infer the original from the surrounding text, and never ask the user to paste the unmasked value back. Report the finding by its rule, severity, and location instead.

Reach for the `skarn-audit` skill when the user asks whether a session or a configuration leaked something, or asks for an audit before sharing a repository or a transcript. This extension carries that skill only.

The guard hook in this extension runs in audit mode. It reports the verdict it would have reached and never blocks what the agent does.

If `gemini mcp list` shows the `skarn` server as Disconnected, run `skarn --version` in the same shell. `command not found` means the binary is not installed or is not on the PATH the CLI inherits. Install it as the repository README describes.
