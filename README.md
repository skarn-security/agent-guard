# Skarn guard: agent plugins

The Skarn guard is a pre-execution hook: before your agent runs a shell command, applies a patch, writes a file, fetches a URL, calls an MCP tool, or sends a prompt, the [Skarn](https://getskarn.com/?utm_source=agent-guard-readme&utm_medium=referral&utm_campaign=home&utm_content=intro) detection engine returns deny, ask, or allow. It scans locally, makes no network call, and prints no raw secret.

## Install

Install the `skarn` binary first; every hook calls `skarn guard` on PATH.

```sh
brew install skarn-security/tap/skarn
```

The buttons add the local MCP server; the table installs the guard.

[![Add to Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/install-mcp?name=skarn&config=eyJjb21tYW5kIjoic2thcm4iLCJhcmdzIjpbIm1jcCJdfQ%3D%3D)

[![Add to VS Code](https://img.shields.io/badge/VS_Code-Add_skarn_MCP-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://vscode.dev/redirect/mcp/install?name=skarn&config=%7B%22type%22%3A%22stdio%22%2C%22command%22%3A%22skarn%22%2C%22args%22%3A%5B%22mcp%22%5D%7D)

Both buttons register the server as `skarn` and run `skarn mcp` from your PATH, so install the binary first. If you would rather not install it, the pinned launcher form works with only Node present: [Cursor](https://cursor.com/install-mcp?name=skarn&config=eyJjb21tYW5kIjoibnB4IiwiYXJncyI6WyIteSIsIkBza2Fybi1zZWN1cml0eS9za2FybkAwLjMxLjAiLCJtY3AiXX0%3D) or [VS Code](https://vscode.dev/redirect/mcp/install?name=skarn&config=%7B%22type%22%3A%22stdio%22%2C%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22%40skarn-security%2Fskarn%400.31.0%22%2C%22mcp%22%5D%7D) - it downloads `@skarn-security/skarn@0.31.0` on first run and reuses the npx cache afterward; a newer Skarn needs a newer link.

| Host | Commands | Caveat |
| --- | --- | --- |
| Claude Code | `claude plugin marketplace add skarn-security/agent-guard`<br>`claude plugin install skarn-guard@skarn` | The hooks are live in your next session, not the running one. |
| Codex CLI | `codex plugin marketplace add skarn-security/agent-guard`<br>`codex plugin add skarn-guard-codex@skarn` | Choose "Trust all and continue" at the "Hooks need review" prompt, or the hooks stay off. A hook change asks again. |
| Gemini CLI | `gemini extensions install https://github.com/skarn-security/agent-guard` | The root is itself the Gemini extension, because Google's installer reads the manifest there, and carries the same hooks, skill, and MCP server. Install asks for consent and folder trust; answer both. |
| Antigravity CLI | `git clone https://github.com/skarn-security/agent-guard`<br>`agy plugin install agent-guard/antigravity` | Fails closed; see Safety. |
| Cursor | `mkdir -p ~/.cursor/plugins/local`<br>`git clone https://github.com/skarn-security/cursor-plugin ~/.cursor/plugins/local/skarn` | Listing pending. Cursor rejects a local plugin whose symlink target lies outside `~/.cursor/plugins/local`, so clone there, then run `Developer: Reload Window`. The hooks are macOS and Linux only, and fail open on Windows. |
| Any host, via `skarn setup` | `skarn setup` | Merges the same hooks into the native config of Claude Code, Codex CLI, Cursor, Copilot CLI, Gemini CLI, and Grok Build, but not Antigravity. |

Both guard plugins carry the `skarn-audit` skill. `skarn-recall`, a skill-only plugin in the same marketplace, reconstructs your past work from transcripts: `claude plugin install skarn-recall@skarn`, or `codex plugin add skarn-recall@skarn`.

## Local MCP server

The four guard units declare the same stdio MCP server, started with `skarn mcp`.

```json
{
  "mcpServers": {
    "skarn": {
      "command": "skarn",
      "args": ["mcp"]
    }
  }
}
```

It exposes four read-only tools: `scan_sessions`, `vet_configs`, `list_sessions`, and `session_stats`. Findings are redacted; metadata and aggregates carry no message content. Two content tools, `search_sessions` and `get_session`, appear only under `skarn mcp --enable-recall`, which these declarations omit.

Without the binary, a pinned launcher works:

```json
{
  "mcpServers": {
    "skarn": {
      "command": "npx",
      "args": ["-y", "@skarn-security/skarn@0.31.0", "mcp"]
    }
  }
}
```

Keep the version pinned; `skarn vet` reports the unpinned form as `vet-mcp-unpinned`.

## Audit first, then enforce

Both guard plugins ship in audit mode. Audit reports the would-be verdict and changes nothing. `SKARN_GUARD_LOG=<path>` in the agent's environment logs one redacted record per flagged call. Once the log is clean, change `--guard-mode audit` to `--guard-mode enforce` in the plugin's hook entries. `skarn setup --update --mode enforce` flips only the `skarn guard` hooks in a host's own config, so it leaves these plugins in audit. Enforcement runs on any paid tier; without one the guard stays in audit, so a lapsed license does not break your editor.

## Updating

```sh
claude plugin marketplace update skarn
claude plugin update skarn-guard@skarn
```

```sh
codex plugin marketplace upgrade
codex plugin add skarn-guard-codex@skarn
```

Plugin and binary updates are independent.

## Safety

The guard only tightens a decision. Claude Code, Codex CLI, and Gemini CLI fail open: a guard error yields allow, unless a licensed enforce runs `--strict`, which also gates `Read`. The Antigravity CLI fails closed on every outcome except an explicit allow, in audit as well as enforce: a crash, a timeout, invalid JSON, or a missing `skarn` binary blocks the matched call. Install skarn before wiring that hook, and remove the entry before uninstalling it.

Transcript content the recall skill reads leaves your machine for the model provider serving the agent; redaction masks the credentials Skarn detects, not everything you would call sensitive.

## License, privacy, and support

This repository is MIT licensed and carries configuration only; see LICENSE. The `skarn` binary is a separate, closed-source download under the Skarn End User License Agreement at https://getskarn.com/terms/, which running it accepts. What it reads and what stays on your machine: https://getskarn.com/privacy/. Support: hello@getskarn.com. Vulnerability reports: security@getskarn.com.
