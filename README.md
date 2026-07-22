# Skarn guard: agent plugins

Hook configuration for [Skarn](https://getskarn.com/?utm_source=agent-guard-readme&utm_medium=referral&utm_campaign=home&utm_content=intro)'s real-time guard, packaged as a native marketplace for Claude Code and Codex CLI. This repository is generated at every Skarn release and holds configuration only: two plugin manifests, two hook files, and this page. It contains no binary, no detection rules, and no engine code.

The guard is a pre-execution hook. Before your agent runs a shell command, applies a patch, writes a file, fetches a URL, calls an MCP tool, or sends a prompt to the model provider, the pending action is scanned by the Skarn detection engine on your machine and gets a verdict: deny (block), ask (escalate to you), or allow (silent). It scans locally, makes no network call, and never prints a raw secret - a verdict names the rule and a redacted preview.

## Install

Install the skarn binary first (the plugin does not carry it; the hooks call `skarn guard` on your PATH):

```sh
brew install skarn-security/tap/skarn
```

Claude Code:

```sh
claude plugin marketplace add skarn-security/agent-guard
claude plugin install skarn-guard@skarn
```

Codex CLI:

```sh
codex plugin marketplace add skarn-security/agent-guard
codex plugin add skarn-guard-codex@skarn
```

The two plugins carry different names because one marketplace cannot hold two plugins under the same name, and each ships its host's hook shape (event names, tool matchers, and the `--agent` the guard routes on). Install the one for the host you are wiring.

Codex asks you to trust the hooks the first time you start it after installing: choose "Trust all and continue" at the "Hooks need review" prompt. Until you do, the hooks do not run and the guard is not protecting you. Trust is a content hash per hook, so any later change to a hook (a version bump, or the audit-to-enforce flip) asks again. Claude Code needs no trust step; the hooks are live in your next session.

`skarn setup` is the alternative install path: it merges the same hooks into each host's native config without the plugin system, and covers Cursor as well.

## Audit first, then enforce

Both plugins ship in **audit** mode: the guard reports the verdict it would have returned and never changes what your agent does. Run it that way first and measure your real would-block rate. Set `SKARN_GUARD_LOG=<path>` in the environment your agent inherits and the guard appends one redacted JSONL record per flagged call (`ts`, `verdict`, `tool`, `rule`, `severity`, redacted reason, `session`, `cwd`, `latency_ms`); clean calls are not logged.

When the log looks clean, flip the `--guard-mode audit` in the hook command to `--guard-mode enforce`, or run `skarn setup --update --mode enforce`. Enforcement requires a Skarn license; without one the guard stays in audit and reports rather than blocks, so a lapsed license never breaks your editor.

## Updating

Claude Code:

```sh
claude plugin marketplace update skarn
claude plugin update skarn-guard@skarn
```

Codex CLI (it has no update verb; re-adding installs the new version):

```sh
codex plugin marketplace upgrade
codex plugin add skarn-guard-codex@skarn
```

Plugin updates and binary updates are independent: the hook commands invoke `skarn guard` on PATH, so `brew upgrade skarn` alone gives you the newer detection engine. A plugin update only matters when the hook shape changes (a new event, a new flag, a new matcher).

## Safety

A malformed event, an out-of-scope tool, or any guard error yields allow: the guard never bricks your agent. It only ever tightens a decision, never loosens one. Documentation: https://getskarn.com/?utm_source=agent-guard-readme&utm_medium=referral&utm_campaign=home&utm_content=documentation
