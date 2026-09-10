# Installing the Skarn MCP server

Work through these sections in order.

## Install the binary

The MCP server is the `skarn` command; this repository carries no binary. Install it however suits the machine:

```sh
brew install skarn-security/tap/skarn
```

```sh
npm install -g @skarn-security/skarn
```

Or download the platform's release binary from https://github.com/skarn-security/skarn-dist/releases and put it on the PATH. The `.tar.gz` extracts a `skarn` executable that does not carry the archive's platform name. On Windows the asset is a bare `skarn-<arch>-windows.exe` with no archive; rename it to `skarn.exe` before putting it on the PATH.

Before extracting or running a download, fetch `SHA256SUMS` and `SHA256SUMS.sigstore.json` from the same release, then verify the signature with [cosign](https://github.com/sigstore/cosign/releases) and the asset against the verified manifest. Replace `<asset>` with the filename as the release published it, so a name the manifest does not carry fails rather than passing silently. On Windows that is the pre-rename `skarn-<arch>-windows.exe`:

```sh
cosign verify-blob SHA256SUMS --bundle SHA256SUMS.sigstore.json \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  --certificate-identity-regexp '^https://github.com/skarn-security/skarn/\.github/workflows/(release|publish-npm)\.yml@'
grep ' <asset>$' SHA256SUMS | shasum -a 256 -c
```

The signature ties the manifest to the Skarn release workflow; without cosign, a matching checksum proves only that the asset and manifest agree, so tell the user which check ran. These are POSIX commands; on Windows, run them in Git Bash.

## Verify the install

```sh
skarn --version
```

If this prints `command not found`, the binary is not installed or is not on the PATH the client inherits. A desktop application often has a shorter PATH than a terminal.

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

In Cline, merge it into `~/.cline/data/settings/cline_mcp_settings.json`, creating the file if it does not exist; both the VS Code extension and the CLI read that one file, and the Configure MCP Servers panel does the same. Cline does not read `~/.cline/mcp.json`.

If the client's PATH does not resolve `skarn`, find the absolute path with `command -v skarn` (POSIX), `(Get-Command skarn).Source` (PowerShell), or `where skarn` (cmd.exe), and replace the command value:

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

If the user prefers not to install the binary, this version-pinned launcher needs only Node:

```json
{
  "mcpServers": {
    "skarn": {
      "command": "npx",
      "args": ["-y", "@skarn-security/skarn@0.30.0", "mcp"]
    }
  }
}
```

The launcher downloads the pinned package on first start, so that start is slower and needs network access; later starts reuse the npx cache. Update the pinned string for a newer Skarn release; an unpinned form resolves to whatever the registry serves, and `skarn vet` reports it as `vet-mcp-unpinned`.

## What the server does

It runs on the user's machine over stdio, makes no network call, and exposes four read-only tools:

- `scan_sessions`: findings from AI coding sessions, with every previewed value redacted.
- `vet_configs`: a masked report on the assistant configuration: hooks, MCP servers, and permission grants.
- `list_sessions`: session metadata: ids, assistant, timestamps, counts. No message content.
- `session_stats`: aggregates. No message content.

Redaction masks detected credential values, and the context around a finding is still session-derived, so treat each result as session data rather than safe text, and do not reconstruct a masked value.

The two tools that return transcript text, `search_sessions` and `get_session`, appear only under `skarn mcp --enable-recall`, which the declaration above omits. Both mask only the credentials the detection rules recognize; prose, code, file paths, personal data, and unrecognized secrets come back as recorded, and all of it leaves the machine for the model provider serving the session. `search_sessions` refuses a call naming neither `hours` nor `project`, and `get_session` takes one exact `session_id` as its scope. Add the flag only when the user asks for session search and understands where the content goes.

On first run, Skarn prints a one-line stderr notice: running it accepts the End User License Agreement at https://getskarn.com/terms/. It does not prompt or block. `skarn eula accept` records acceptance; `SKARN_EULA_ACCEPTED=1` in the environment does the same without writing anything.
