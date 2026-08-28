# Installing the Skarn MCP server

This file is for an agent setting Skarn up on a user's machine. Work through the sections in order.

## Install the binary

This repository carries no binary. The MCP server is the `skarn` command, so install it first. Use whichever of these fits the machine:

```sh
brew install skarn-security/tap/skarn
```

```sh
npm install -g @skarn-security/skarn
```

Or download the release binary for the platform from https://github.com/skarn-security/skarn-dist/releases and put it on the PATH. The `.tar.gz` extracts a `skarn` executable beside the man pages and the licence files; the executable is what goes on the PATH, and it does not carry the archive's platform name. On Windows the asset is a bare `skarn-<arch>-windows.exe` with no archive; rename it to `skarn.exe` before putting it on the PATH.

Verify the download before extracting or running it. Fetch `SHA256SUMS` and `SHA256SUMS.sigstore.json` from the same release, check the signature with [cosign](https://github.com/sigstore/cosign/releases), then check the downloaded asset against the verified manifest, with `<asset>` replaced by the filename as the release published it (on Windows that is `skarn-<arch>-windows.exe`, verified before the rename) so a name the manifest does not carry fails rather than passing silently:

```sh
cosign verify-blob SHA256SUMS --bundle SHA256SUMS.sigstore.json \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  --certificate-identity-regexp '^https://github.com/skarn-security/skarn/\.github/workflows/(release|publish-npm)\.yml@'
grep ' <asset>$' SHA256SUMS | shasum -a 256 -c
```

The signature is what ties the manifest to the Skarn release workflow. Without cosign, a matching checksum shows only that the asset and the manifest agree with each other, so tell the user which of the two checks ran. These are POSIX commands; on Windows run them in Git Bash, where cosign's Windows build works the same way.

## Verify the install

```sh
skarn --version
```

If this prints `command not found`, the binary is not installed or is not on the PATH the client inherits. An application started from the desktop rather than from a shell often has a shorter PATH than a terminal does.

## Declare the server

Add this to the client's MCP settings:

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

In Cline, merge it into `~/.cline/data/settings/cline_mcp_settings.json` (create the file with this object if it does not exist; both the VS Code extension and the CLI read that one file), or paste it into the Configure MCP Servers panel. Cline does not read `~/.cline/mcp.json`. Other clients take the same object under their own MCP settings key.

If the client's inherited PATH does not resolve `skarn`, declare the absolute path instead. Find it from a terminal with `command -v skarn` (POSIX shells), `(Get-Command skarn).Source` (PowerShell), or `where skarn` (cmd.exe), and put the exact output in place of the placeholder:

```json
{
  "mcpServers": {
    "skarn": {
      "command": "/absolute/path/to/skarn",
      "args": ["mcp"]
    }
  }
}
```

A Windows path in JSON needs each backslash doubled:

```json
{
  "mcpServers": {
    "skarn": {
      "command": "C:\\absolute\\path\\to\\skarn.exe",
      "args": ["mcp"]
    }
  }
}
```

## The launcher alternative

If the user would rather not install the binary, a version-pinned launcher form starts the server with only Node present:

```json
{
  "mcpServers": {
    "skarn": {
      "command": "npx",
      "args": ["-y", "@skarn-security/skarn@0.27.0", "mcp"]
    }
  }
}
```

This downloads the pinned package the first time the server starts, so the first start is slower and needs network access, and reuses the npx cache afterwards. A newer Skarn needs a newer version in that string. Keep the version pinned. The unpinned form resolves to whatever the registry serves at start time, and `skarn vet` reports it as `vet-mcp-unpinned`.

## What the server does

It runs on the user's machine over stdio and makes no network call. It exposes four read-only tools, and no tool that writes:

- `scan_sessions`: findings from the AI coding sessions on this machine, every previewed value redacted.
- `vet_configs`: a masked report on the assistant configuration surface, covering hooks, MCP servers, and permission grants.
- `list_sessions`: session metadata, being ids, assistant, timestamps, and counts. Never message content.
- `session_stats`: aggregates over those sessions. Never message content.

Every result is redacted before it leaves the process. Redaction masks the credential values Skarn detects; the surrounding transcript context in a finding is still session-derived, so treat a whole result as session data rather than as safe text, and never try to reconstruct a masked value.

None of those four returns transcript text. Two tools that do, `search_sessions` and `get_session`, appear only when the server is started as `skarn mcp --enable-recall`, which the declaration above deliberately does not do. Both mask what they return with the same redactor the export command uses, but the content they return still leaves the machine for whichever model provider serves the session. `search_sessions` refuses a call that names neither `hours` nor `project`, so it can never return an unbounded sweep; `get_session` takes one exact `session_id` and an optional `view`, and rejects `hours` or `project` as unknown keys, because the id is already the scope. Add the flag only when the user asks for session search and understands where the content goes.

On its first run Skarn prints a one-line notice on stderr saying that running it accepts the End User License Agreement at https://getskarn.com/terms/. It does not prompt and it does not block, so the server starts normally. `skarn eula accept` records acceptance and silences the notice, and `SKARN_EULA_ACCEPTED=1` in the environment the client starts the server with does the same without writing anything.
