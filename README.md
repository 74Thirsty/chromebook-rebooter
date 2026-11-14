<!--
Copyright © 2026 C.E. Hirschauer.
Chromebook Rebooter is an independent project and is not affiliated with, endorsed by, or sponsored by Google LLC.
-->

<p align="center">
  <img src="assets/chromebook-rebooter-animated.svg" alt="Chromebook Rebooter — Frankenstein-style green recovery mascot surrounded by animated electrical effects" width="100%" />
</p>

<h1 align="center">Chromebook Rebooter</h1>

<p align="center">
  <strong>Rebuild. Restore. Reboot.</strong><br />
  An independent Linux desktop application for creating ChromeOS recovery media.
</p>

<p align="center">
  Identify a Chromebook model, retrieve its matching official recovery image, verify the download, and create recovery USB media through a guided desktop workflow.
</p>

> **Project status:** Development is beginning. Do not treat this repository as a released or production-ready recovery tool until the release requirements and hardware validation are complete.

## Product goals

- Search and identify Chromebook models and board names.
- Retrieve recovery image information from Google's published recovery catalog.
- Download and verify recovery images before writing.
- Detect removable drives and protect the host system disk.
- Guide users through an explicit, safety-checked USB writing process.
- Verify written media and provide useful diagnostics.
- Ship as a branded, installable Debian `.deb` package.

## Important safety notice

Creating recovery media is separate from using it to recover a Chromebook. Using recovery media on a Chromebook can erase local data on that Chromebook. Chromebook Rebooter does not remove enterprise enrollment, MDM, ownership, firmware locks, or organizational management.

## Project ownership and notices

Copyright © 2026 C.E. Hirschauer. Project-authored code and original project materials are credited to C.E. Hirschauer, subject to the project's license and all applicable third-party rights.

The Android robot is reproduced or modified from work created and shared by Google and used according to terms described in the Creative Commons 3.0 Attribution License. Android is a trademark of Google LLC. Google, Chromebook, and ChromeOS are trademarks of their respective owners. Google retains trademark rights to the Android robot and its derivatives.

**Chromebook Rebooter is an independent project and is not affiliated with, endorsed by, or sponsored by Google LLC.**

Third-party source code, dependencies, and assets retain their respective owners' copyrights and license terms. See [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) when available.

## Engineering directive

The authoritative implementation scope, security requirements, validation plan, and release acceptance criteria are in [Executive Engineering Directive CR-001](docs/EXECUTIVE_ENGINEERING_DIRECTIVE.md).

## License

The project license and third-party notices must be reviewed before the first public release. See the repository's license and notice files.
