# Security policy

## Reporting a vulnerability

Report suspected vulnerabilities privately, not in a public issue. Use GitHub's private vulnerability reporting on this repository (the Security tab, "Report a vulnerability"), or the contact on https://getskarn.com. Expect an acknowledgement within one business day.

## Scope

This repository is the Skarn plugin distribution: three plugin manifests, two hook configuration files, and three skill files for Claude Code and Codex CLI. It does not contain the Skarn detection engine, its rules, or the binary - those are installed separately and maintained on https://getskarn.com, where vulnerabilities in the scanner itself are handled. The hook files here declare which agent events are intercepted and which command is run; they execute nothing on their own. The recall skill reads session content when invoked.
