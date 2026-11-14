# Security Policy

## Project status

Chromebook Rebooter is under active development and is not yet a released or production-ready recovery tool. Do not use development builds to write recovery media on systems containing important data.

## Reporting a vulnerability

Please do not disclose suspected vulnerabilities in public issues, discussions, or pull requests.

Use GitHub's **Report a vulnerability** feature on this repository to submit a private vulnerability report. Include:

- A concise description of the issue and its potential impact.
- The affected commit, version, component, or workflow.
- Reproduction steps or a proof of concept, where safe.
- Any relevant logs with credentials, tokens, personal data, and other secrets removed.

If GitHub's private vulnerability reporting feature is unavailable, contact the repository owner privately through GitHub before sharing technical details publicly.

## Response and remediation

The maintainers will assess reports, work to reproduce and scope the issue, and coordinate remediation and disclosure with the reporter. No response or remediation deadline is guaranteed while the project is in development.

## Scope

Security reports are especially important for:

- Recovery-image source authenticity, download integrity, and verification.
- Removable-drive detection and protection of host/system disks.
- Privilege escalation, privileged helper processes, and USB-writing operations.
- Unsafe archive or image handling.
- Dependency and build/release pipeline compromise.
- Exposure of credentials, tokens, or sensitive local data.

## Supported versions

No stable release is currently designated as supported. The development branch is the current review target; this does not imply that development builds are safe for production use.
