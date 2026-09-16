# Security policy

## Reporting a vulnerability

Report suspected vulnerabilities privately, not in a public issue. Use GitHub's private vulnerability reporting on this repository (the Security tab, "Report a vulnerability"), or the contact on https://getskarn.com. Expect an acknowledgement within one business day.

## Scope

This repository is the Skarn plugin distribution for Claude Code, Codex CLI, Gemini CLI, and the Antigravity CLI. It carries five units: the Claude Code plugin in `claude/`, the Codex CLI plugin in `codex/`, the Antigravity CLI plugin in `antigravity/`, the Gemini CLI extension at the repository root, and the skill-only `skarn-recall` plugin in `recall/`, plus the marketplace catalog that lists the Claude Code, Codex CLI, and recall plugins. Each of the four guard units carries a manifest, a hook configuration file, an MCP server declaration, and the `skarn-audit` skill; the Antigravity CLI unit carries a second hook configuration file for enforce mode. The recall unit carries a manifest and the `skarn-recall` skill only. The Cursor plugin is published separately from https://github.com/skarn-security/cursor-plugin, and `skarn setup` is part of the binary.

This repository does not contain the Skarn detection engine, its rules, or the binary - those are installed separately and maintained on https://getskarn.com, where vulnerabilities in the scanner itself are handled. The hook files here declare which agent events are intercepted and which command is run; they execute nothing on their own. The MCP declarations name a command to spawn; they start nothing on their own. The recall skill reads session content when invoked.
