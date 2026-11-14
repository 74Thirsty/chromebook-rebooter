# EXECUTIVE ENGINEERING DIRECTIVE

**Directive ID:** CR-001  
**Title:** Chromebook Rebooter — Complete Product, Packaging, and Release  
**Owner:** C.E. Hirschauer  
**Repository:** `74Thirsty/chromebook-rebooter`  
**Priority:** P0 — Product completion  
**Status:** AUTHORIZED — BUILD TO RELEASE  
**Target release:** A tested, installable Linux `.deb` package  
**Decision authority:** Project owner

---

## 1. Executive objective

Build **Chromebook Rebooter** into a complete, polished, dependable Linux desktop application that guides an ordinary user through creating official ChromeOS recovery USB media.

This is not a prototype, mockup, proof of concept, CLI wrapper, or documentation-only delivery. The required outcome is a finished application with a branded graphical interface, working recovery-image discovery and download, integrity verification, safe USB writing, clear progress and recovery from errors, automated tests, installation packaging, and a release-ready Debian package.

The application must make the correct workflow understandable to a novice without hiding the destructive nature of writing a USB drive. The experience should be guided and difficult to misuse—not marketed as literally risk-free.

**Definition of success:** a new user can install the released `.deb`, launch the branded application from the desktop, identify or select a Chromebook model, obtain the matching recovery image from the official recovery catalog, verify it, select the intended removable drive, confirm the destructive operation with unambiguous device details, write and verify the media, and receive a clear completion result.

Do not declare completion because the window opens or a USB write command exits zero. Completion requires the acceptance evidence in this directive.

## 2. Executive engineering directive

### STATUS
- Repository exists and is public.
- Root-level `assets/` directory exists.
- Existing brand assets include:
  - `assets/chromebook-rebooter-animated.svg`
  - `assets/Frankenstein Android Mascot Render.png`
- The animated SVG is the required README hero artwork.
- Product owner is C.E. Hirschauer.

### PHASE
Inspect → architecture and licensing audit → implement → test → package → install-test → hardware validation → release.

### PRIORITY
1. Correct recovery-image selection and provenance.
2. Prevent accidental writes to internal or incorrect disks.
3. Reliable image download, integrity checking, writing, and post-write verification.
4. Complete novice-friendly workflow and error handling.
5. Branded desktop experience and accessibility.
6. Reproducible `.deb` build, installation, upgrade, and removal.
7. Documentation, legal notices, and release evidence.

### ALLOWED
- Reuse compatible open-source code and libraries after checking their actual licenses and obligations.
- Use Google's published ChromeOS recovery metadata and official image hosting endpoints.
- Study and, where legally compatible and technically appropriate, adapt Google's published Linux recovery implementation.
- Refactor or replace implementation choices when testing demonstrates a better supported approach.
- Add tests, CI, packaging, documentation, release automation, and required assets.
- Add robust features that directly improve recovery-media creation, safety, usability, diagnostics, or maintainability.

### PROHIBITED
- Do not invent recovery URLs, board mappings, checksums, image versions, device capabilities, or test results.
- Do not silently select an image when the board/model match is ambiguous.
- Do not write to a block device without a fresh device identity check and explicit final user confirmation.
- Never offer the host system disk, its parent disk, or a disk containing a mounted root/boot filesystem as a normal write target.
- Never use shell string interpolation for privileged commands or pass arbitrary commands through a root helper.
- Never run the entire GUI as root.
- Do not claim to remove enterprise enrollment, MDM, device ownership, firmware locks, or other organization controls. Recovery media creation is not an enrollment-removal feature.
- Do not imply Google endorsement, affiliation, or official status.
- Do not copy third-party source or assets without preserving required license notices and attribution.
- Do not claim a physical-device test passed unless it was actually run and recorded.
- Do not stop at a working development launch; the required deliverable includes a working, installable `.deb`.

---

## 3. Initial repository inspection — mandatory first task

Before implementing, inspect the complete repository and report:
- Current branch, commit, working tree state, and repository structure.
- Existing README and every file under `assets/`.
- Whether the SVG is valid, self-contained, animated in supported renderers, and suitable for a GitHub README.
- Whether the SVG embeds raster data, references external files, uses scripts, or contains external network dependencies.
- Existing source, build configuration, tests, CI, license files, and packaging (if any).
- Exact filenames and dimensions of all icon/brand assets.

Do not overwrite or delete existing files merely because they appear unused. Preserve the user's assets. If the SVG is too large or GitHub sanitization prevents reliable animation, retain it as the full hero asset and create a separately optimized static hero/icon variant only if needed. Do not silently substitute another image.

The repository is currently a clean product start. Establish a maintainable structure based on the implementation actually selected; do not create empty scaffolding that is never used.

## 4. Product definition and user workflow

The application creates **ChromeOS recovery media**. It does not install Linux, create Shimboot media, unlock a Chromebook, or alter enterprise enrollment.

The main workflow must be a guided wizard:

1. **Welcome**
   - Branded welcome screen and concise explanation.
   - Explain that the USB drive will be erased.
   - Explain that creating recovery media is separate from using it to recover a Chromebook.
2. **Identify Chromebook**
   - Searchable manufacturer/model catalogue.
   - Search by model name, board name, HWID where available, and aliases.
   - Allow manual board entry only with validation and a clear uncertainty warning.
   - Display the exact selected board/model and how the match was obtained.
   - Do not pretend the Linux host can automatically identify a separate Chromebook that is not connected.
   - If a connected target device can genuinely be queried through a supported interface, make that optional and explain the source; never infer identity from a USB VID/PID alone.
3. **Choose recovery image**
   - Retrieve current official metadata from Google's published recovery catalog.
   - Present only images actually returned by the catalog and matched to the selected device.
   - Show model/HWID/board identifiers, channel, version, image filename, published size, and checksum availability where supplied.
   - Distinguish stable/recommended from other channels only when the metadata supports that distinction.
   - Show metadata retrieval time and source.
   - If no match exists, explain that plainly and provide a safe manual refresh/retry path. Do not substitute a similar board.
4. **Download**
   - Show determinate progress when the server supplies reliable size information; otherwise show honest indeterminate progress.
   - Support cancellation and safe resume where HTTP range support and local partial-file validation allow it.
   - Cache completed downloads and metadata safely.
   - Avoid duplicate downloads and detect stale/partial cache entries.
5. **Verify**
   - Verify downloaded archive size and the strongest checksum supplied by trusted metadata.
   - Prefer SHA-256 when the source provides it; support legacy SHA-1/MD5 only when required by Google's actual metadata, clearly label the algorithm, and do not describe weak legacy hashes as modern cryptographic assurance.
   - Verify the expected archive member and extracted image size.
   - Reject path traversal, unexpected archive members, corrupt archives, truncated files, and malformed metadata.
   - Do not write an image whose integrity check failed or whose trusted metadata lacks sufficient validation for the implemented policy.
6. **Select USB drive**
   - Detect and refresh removable storage using reliable Linux device information (for example UDisks2, udev, sysfs, and block-device metadata).
   - Show device path, vendor/model, capacity, transport, serial where available, and mounted partitions.
   - Explain why a device is excluded.
   - Check required capacity before enabling the next step.
   - Exclude the system disk, its parent, disks backing mounted root/boot/swap, loop devices, optical devices, and unsuitable devices.
   - Do not rely on a single `removable=1` flag; USB bridges and SD readers vary.
   - Permit a user to refresh devices and revalidate the selected device immediately before writing.
7. **Final destructive-operation confirmation**
   - Show a prominent warning that all data on the selected drive will be destroyed.
   - Display the exact device path, make/model, capacity, and serial (when available).
   - Require a deliberate confirmation action; a typed confirmation containing the device path is preferred for the final release.
   - If the selected device disappears, changes identity, changes capacity unexpectedly, or becomes a protected/system disk, abort.
8. **Write**
   - Unmount all mounted child partitions cleanly.
   - Acquire only the privilege required for the raw-device write.
   - Use a narrow, auditable privileged helper or UDisks2-supported authorization flow. Never elevate the full GUI.
   - Revalidate the target identity after authorization and immediately before opening it.
   - Stream the image with bounded memory, progress reporting, cancellation semantics, robust short-write handling, and clear error propagation.
   - Flush buffers and call the appropriate sync/fsync/block-device flush operations.
   - Never report success before all write and verification steps finish.
9. **Post-write verification and completion**
   - Verify written data against the source image by read-back or an equally strong, documented verification strategy.
   - If full read-back verification is not practical on a device, explicitly disclose the reduced assurance; do not silently skip it.
   - Safely eject/power off when supported.
   - Show success only after verification, with model, image version/channel, target drive, and elapsed time.
   - Explain how to use the recovery USB and warn that recovery on the Chromebook can erase local device data.
10. **Diagnostics**
    - Provide a copy/export diagnostics action.
    - Logs must not include secrets, credentials, or unnecessary personal data.
    - Include actionable errors, not raw stack traces as the only user-facing message.

---

## 5. Recovery metadata and upstream implementation

Use Google's official recovery catalog as the primary source of image metadata. The legacy Linux recovery implementation in ChromiumOS's `user-recovery-tools` repository is a valuable reference and potential source of reusable code:

- Upstream Linux recovery implementation: https://chromium.googlesource.com/chromiumos/user-recovery-tools/+/master/linux/recovery.sh
- Official recovery catalog endpoint used by that implementation: https://dl.google.com/dl/edgedl/chromeos/recovery/recovery.conf
- Upstream project: https://chromium.googlesource.com/chromiumos/user-recovery-tools/

The upstream script documents a Linux workflow for retrieving the recovery configuration, presenting image choices, downloading and validating an image, extracting it, selecting a removable drive, and writing recovery media. It is a reference implementation, not a guarantee that its old assumptions meet this product's safety or UX requirements.

**Required upstream audit:**
- Inspect the repository's actual `LICENSE` and any file-specific notices before copying or adapting code.
- Preserve ChromiumOS Authors' copyright and BSD license notices for any copied/adapted source.
- Prefer a clean, tested implementation of the metadata parser, download manager, and device writer over blindly embedding the legacy shell script.
- If the upstream script is invoked as a subprocess, pin/audit the exact source version, preserve notices, avoid parsing unstable human-readable output, and do not let it control privileged device writes without the safety checks in this directive.
- Treat Google's catalog as external data: validate syntax, fields, sizes, checksums, URLs, and image identifiers. Fail closed on malformed or unexpected data.
- Use HTTPS, validate redirect destinations, and reject non-HTTPS image URLs unless a documented official exception is proven necessary.
- Do not scrape Chrome Web Store extension internals or depend on private undocumented APIs.

Do not assume Google's metadata schema is immutable. Add parser fixtures based on real, captured metadata and tests for missing, duplicate, malformed, old, and newly added fields. The application must display when the official metadata service is unavailable and must not fabricate a local catalogue as if it were current.

## 5A. Related open-source Chromebook projects — permitted research and reuse

The owner explicitly authorizes the engineering agent to investigate relevant open-source Chromebook projects—including **Shimboot** and **sh1mmer** (the project name uses the numeral **1**)—as potential references or sources of reusable components. Do not assume either project is suitable: identify the exact upstream repository, maintainer, current status, source files, license, supported devices, and actual behavior before relying on it.

This authorization means:
- You may study their documented workflows, device-detection methods, recovery/boot-media handling, compatibility information, and reusable non-circumvention components.
- You may adapt or incorporate components only when their licenses permit it, attribution and notices are preserved, the code is maintained and technically appropriate, and it directly supports this product's authorized scope.
- You may document interoperability or explain how a user can separately obtain/use a related tool, provided the instructions are accurate, lawful, and do not misrepresent Chromebook Rebooter as that tool.
- Keep any optional integration isolated, clearly labeled, disabled by default unless explicitly approved, and covered by tests and the same USB safety controls.

**Scope boundary:** Chromebook Rebooter remains a ChromeOS recovery-media creator. Shimboot's separate Linux-boot workflow is not automatically part of this product; do not add a general Linux-boot mode or change the product's purpose without a written owner-approved scope change.

**Security and authorization boundary:** Do not implement, adapt, automate, or provide operational instructions for using sh1mmer or any other exploit/utility to bypass or remove enterprise enrollment, MDM, device ownership, activation, firmware locks, or organization controls. Do not use such a tool to circumvent management on a device. The existing prohibition on MDM/enrollment/ownership bypass remains absolute. If an upstream project combines legitimate recovery functionality with circumvention functionality, isolate and exclude the circumvention portions; if that cannot be done confidently, do not reuse the project.

Before any reuse, add an upstream audit record to `THIRD_PARTY_NOTICES.md` and document the exact repository URL, commit/tag, files reused, license, notices, security review, and tests. If no safe, license-compatible component is identified, report that conclusion and proceed with the native implementation; do not force an integration merely because the project was named.

---

## 6. Technical architecture

Choose a small, supportable architecture that can be packaged reliably on Debian-derived distributions. The preferred direction is:

- **Language/backend:** Python 3 with typed modules and clear service boundaries.
- **Desktop UI:** GTK 4 / PyGObject for native Linux integration and Debian package dependencies.
- **Device discovery and lifecycle:** UDisks2 over D-Bus plus udev/sysfs evidence as needed.
- **Privilege boundary:** polkit-authorized, narrowly scoped helper for raw writes; GUI remains unprivileged.
- **Networking:** maintained HTTP client with timeouts, retry policy, TLS validation, redirect restrictions, and resumable downloads where safe.
- **Metadata/integrity:** strict parser, `hashlib`, archive validation, and typed domain models.
- **Packaging:** Debian-native package structure with `dpkg-buildpackage`/debhelper where practical; include desktop entry, icon theme assets, AppStream metadata, dependencies, maintainer scripts only when necessary, and clean uninstall behavior.

The agent may propose a different stack only after inspecting the repository and documenting concrete advantages, Debian packaging impact, security implications, and testability. Do not change stacks for novelty.

Suggested modular boundaries (adapt to the actual code):
- `catalog/`: metadata retrieval, parsing, validation, model/board matching.
- `downloads/`: cache, resume, progress, cancellation.
- `verification/`: checksums, archive safety, image validation, read-back verification.
- `devices/`: enumeration, safety classification, revalidation, unmount/eject.
- `writer/`: narrow privileged writer and progress protocol.
- `ui/`: wizard, reusable components, accessibility, error presentation.
- `packaging/`: Debian metadata, icons, desktop entry, AppStream, release scripts.
- `tests/`: unit, integration, packaging, and hardware test documentation.

Keep business logic out of UI callbacks. Use typed data models, structured logging, explicit error types, deterministic state transitions, and cancellation that cannot leave the UI claiming success while a write is still active.

## 7. Safety and security requirements

USB writing is destructive. Safety is a primary product feature, not a disclaimer added at the end.

Mandatory controls:
- Never write to a path supplied only by stale UI state.
- Resolve and record the selected block device's stable identity (major/minor, sysfs path, model, serial where available, capacity, and parent relationship).
- Re-enumerate and compare identity immediately before write.
- Detect system/root/boot/swap backing devices and all their parent disks; hard-block them.
- Hard-block devices that are not suitable block devices.
- Require a final explicit confirmation after showing the exact target.
- Use safe argument arrays, not shell command strings.
- Keep privileged code minimal, separately testable, and auditable.
- Validate all input paths and metadata; protect against archive path traversal and symlink attacks.
- Store cache/log files with safe permissions and avoid following untrusted symlinks.
- Handle device removal, disconnects, power loss, low disk space, network loss, checksum mismatch, and write/read-back errors.
- Do not automatically retry a destructive write after an uncertain partial failure. Explain the state and require a fresh review.
- Do not silently reformat or alter the host's disks.
- Provide a clear abort path before writing; during writing, cancellation must be handled safely and honestly.

The application must not offer MDM removal, enrollment bypass, firmware lock circumvention, or ownership bypass. State clearly that creating recovery media does not release enterprise-managed devices. The legitimate owner/administrator must release management through the appropriate organization process.

## 8. Branding, assets, and ownership

Branding is part of the release, not a later polish task.

Required:
- Product name: **Chromebook Rebooter**.
- Use the animated SVG in the README hero: `assets/chromebook-rebooter-animated.svg`.
- Use the Frankenstein-style green mascot as the recognizable application identity.
- Create/derive a complete icon set from the supplied mascot for desktop launchers, window icon, About dialog, installer/package metadata, and application assets.
- Preserve the original source assets. Do not destructively overwrite the supplied PNG or SVG.
- Generate crisp square icon sizes (at minimum 16, 24, 32, 48, 64, 128, 256, and 512 px) and a scalable icon if technically feasible.
- Provide a static icon for environments that do not animate icons; do not rely on animated SVG support for the desktop icon.
- Use the animated SVG in places where animation is supported and performance is appropriate.
- Respect reduced-motion preferences and avoid high-frequency flashing. Lightning effects must not create photosensitive seizure risks.
- Use consistent typography, spacing, green/charcoal palette, polished empty/loading/error/success states, and accessible contrast.
- Include an About dialog with product name, version, author, project URL, license, and third-party notices.

### Copyright and attribution

For project-authored source code and original project materials, use the owner credit:

**Copyright © 2026 C.E. Hirschauer.**

Preserve third-party notices for all reused source, dependencies, fonts, and artwork. Do not put the owner's copyright over Google's source code or third-party materials.

The mascot is visibly based on a modified Android robot. Google's current Android brand guidance states that the Android robot may be reproduced or modified under Creative Commons Attribution 3.0, with the required attribution, and that Google retains trademark rights to the robot and its derivatives. Include this attribution in the repository's third-party notices and in an accessible credits/legal section:

> The Android robot is reproduced or modified from work created and shared by Google and used according to terms described in the Creative Commons 3.0 Attribution License.

Also state that Android is a trademark of Google LLC where the Android name/robot is referenced. Do not claim exclusive trademark ownership over the Android robot or any derivative. The project's own contribution/branding credit to C.E. Hirschauer must remain distinct from Google's attribution.

Use a clear non-affiliation notice: **Chromebook Rebooter is an independent project and is not affiliated with, endorsed by, or sponsored by Google LLC. Google, Chromebook, ChromeOS, and Android are trademarks of their respective owners.** Verify the exact wording against current brand guidance before release.

The project owner wants proper copyright credit. Do not omit the C.E. Hirschauer copyright notice from project-authored code headers where appropriate, README, About dialog, and release metadata. If the project license is MIT, include the copyright line in the MIT `LICENSE`; retain all third-party licenses separately. Do not silently select a license if the repository already contains a different owner-approved license. If no license exists, propose MIT for owner confirmation and record the decision before publishing a release.

Official Android brand guidance:
- https://developer.android.com/distribute/marketing-tools/brand-guidelines
- https://partnermarketinghub.withgoogle.com/brands/android/legal-and-trademarks/

## 9. Feature-rich product expectations

Beyond the essential happy path, implement the following where they can be completed reliably:

- Searchable model/board catalog and aliases.
- Model/board detail and source provenance.
- Metadata refresh with last-updated indicator.
- Recovery channel/version selection when official metadata supports it.
- Download progress, cancel, retry, resume, and cache management.
- Checksum and extracted-image verification.
- Live removable-drive detection and refresh.
- Drive capacity validation and protected-drive explanations.
- Final destructive confirmation with exact device identity.
- Write progress, throughput, estimated remaining time where meaningful.
- Post-write verification and safe eject.
- Clear recovery from network, disk-space, archive, checksum, USB, permission, and write failures.
- Diagnostics export/copy with privacy-conscious redaction.
- First-run explanation and concise in-app help.
- About dialog, version information, copyright and third-party credits.
- Keyboard navigation, screen-reader labels, high-contrast readability, and reduced-motion support.
- Offline access to previously downloaded, verified images; clearly indicate that catalog freshness cannot be checked offline.
- Settings for download/cache location and diagnostics verbosity, with safe defaults.
- A preflight checklist before writing.

Do not add unrelated features such as general-purpose disk partitioning, OS installation, firmware modification, MDM removal, or a generic multi-OS flashing suite. Keep the product focused.

## 10. Testing and validation

All tests must be reproducible and evidence-backed. Add tests before relying on a feature.

### Unit tests
- Recovery metadata parser and schema validation.
- Board/model/HWID matching, aliases, ambiguous matches, and no-match cases.
- URL and redirect validation.
- Download retry/resume and partial-file cleanup.
- Archive path traversal and malformed archive rejection.
- Checksum mismatch, size mismatch, and missing-checksum policy.
- Device classification and protected-disk ancestry.
- Device identity changes between selection and write.
- Progress/state-machine transitions and cancellation.
- Privilege helper input validation and authorization failure handling.

### Integration tests
- Local HTTP fixture server for metadata and image downloads.
- Small synthetic image written to a loopback device or regular test file through a test-only writer abstraction.
- Read-back verification and injected short writes/errors.
- USB removal and simulated permission failures.
- Cache recovery after interrupted downloads.
- UI wizard tests for success, no-match, catalog offline, checksum failure, no USB, unsafe USB, insufficient capacity, and write failure.

**Never use a real host disk as a test fixture.** Physical-device tests must use a dedicated sacrificial USB drive and must be clearly documented.

### Packaging tests
- Build the `.deb` from a clean checkout.
- Inspect package metadata, dependency declarations, file ownership, desktop entry, icon, AppStream metadata, and license notices.
- Install into a clean Debian-derived VM/container with supported GUI dependencies.
- Launch from the desktop menu, verify About/version/icon, perform a non-destructive catalog lookup, and verify clean uninstall.
- Test upgrade from the previous package version once one exists.
- Confirm no files are installed into a user's home directory without consent.
- Verify the application does not require running as root.

### Hardware validation
Before a stable release, test actual USB writing and read-back verification on at least two distinct removable drives and actual Chromebook recovery-media boot on at least one supported Chromebook. Record exact device model, board, image version/channel, host distro, drive make/model/capacity, test date, and result.

The initial target device discussed by the owner is an Acer Chromebook Spin 511 family device associated with the board name the owner identified as `craaskbowl`. Treat this as a test target to verify against official metadata and actual device identifiers; do not hardcode the spelling or assert compatibility until verified. Add it to the supported/tested matrix only after evidence is collected.

If physical Chromebook testing is unavailable, the release must be labeled **beta / hardware validation pending**, not production-certified.

## 11. Debian release requirements

A working `.deb` is mandatory.

Deliver:
- Versioned `.deb` package for the supported initial architecture (at minimum amd64).
- Debian package metadata with correct name, version, architecture, dependencies, maintainer, description, and homepage.
- Desktop launcher and correct application icon.
- AppStream metainfo.
- Clean installation, upgrade, and removal behavior.
- No root-owned files in user home directories.
- No hard-coded developer paths or machine-specific configuration.
- Reproducible build instructions and a single documented build command.
- CI workflow that runs tests and builds the package.
- Release notes and SHA-256 checksum for the published package.
- A GitHub Release containing the `.deb`, checksum, release notes, and supported-system statement.

Do not call a package “ready” until it has been installed and launched in a clean supported environment. Do not publish a stable release with known critical defects or unverified destructive-write behavior.

## 12. Documentation deliverables

Create and maintain:
- `README.md`: animated SVG hero, concise product description, screenshots, features, requirements, installation, usage, safety warning, support, project status, license, copyright, attribution, and non-affiliation notice.
- `docs/ARCHITECTURE.md`: modules, trust boundaries, privilege model, data flow, and key design decisions.
- `docs/TEST_PLAN.md`: unit/integration/package/hardware test matrix and evidence.
- `docs/RELEASE.md`: reproducible build, package validation, signing/checksum, and release procedure.
- `THIRD_PARTY_NOTICES.md`: upstream code, dependency, artwork, font, and trademark/attribution notices.
- `LICENSE`: owner-approved project license, distinct from third-party licenses.
- `CHANGELOG.md`: user-visible changes by release.

Keep documentation synchronized with actual behavior. No fictional screenshots, unsupported feature claims, or fabricated test badges.

## 13. Execution phases and checkpoints

### Phase 0 — Reconnaissance and decision record
Inspect repo, assets, official recovery metadata, upstream source and licenses. Produce a concise report with verified facts, risks, proposed architecture, and any owner decisions required.

**Checkpoint:** no implementation begins on assumptions about the recovery catalog, board mapping, or licenses.

### Phase 1 — Core recovery engine
Implement catalog retrieval/parsing, model matching, downloads, cache, archive validation, checksums, and deterministic error handling.

**Checkpoint:** automated tests pass using captured fixtures and a local test server.

### Phase 2 — Device safety and writer
Implement removable-drive discovery, system-disk protection, identity revalidation, privilege boundary, writing, flush, read-back verification, and safe eject.

**Checkpoint:** loopback/synthetic-image tests pass; protected devices are demonstrably blocked; privileged helper review is complete.

### Phase 3 — Complete branded GUI
Implement the full wizard and all state/error/loading/success paths. Integrate mascot, icon family, About dialog, credits, and accessibility/reduced-motion behavior.

**Checkpoint:** novice workflow can be completed against local fixtures without developer intervention.

### Phase 4 — Packaging and CI
Implement Debian package, desktop integration, AppStream, automated CI, and release documentation.

**Checkpoint:** clean-environment package install/launch/uninstall passes.

### Phase 5 — Hardware validation and release
Run dedicated USB write/read-back tests and Chromebook boot validation. Fix all release blockers. Publish the versioned `.deb` and GitHub Release only when acceptance criteria are satisfied.

**Checkpoint:** final evidence report and owner release approval.

---

## 14. Acceptance criteria — all mandatory

The directive is complete only when all applicable items are independently demonstrated:

- [ ] App launches as a normal desktop user from the installed package.
- [ ] Correct branding, application icon, About dialog, and C.E. Hirschauer copyright credit are present.
- [ ] README hero displays `assets/chromebook-rebooter-animated.svg`.
- [ ] Google/Android attribution, third-party notices, license, and non-affiliation statement are present and accurate.
- [ ] Official recovery metadata can be retrieved, parsed, and refreshed.
- [ ] Search/model/board matching works and ambiguous/no-match cases fail safely.
- [ ] Selected recovery image details and source are visible.
- [ ] Downloads can be canceled and safely resumed where supported.
- [ ] Archive and image integrity checks pass; corrupted or mismatched data is rejected.
- [ ] USB device discovery and refresh work.
- [ ] System disks and protected devices cannot be selected for writing.
- [ ] Selected device identity is revalidated immediately before writing.
- [ ] Final destructive confirmation clearly identifies the exact target.
- [ ] Privileged writing uses a narrow authorization boundary; GUI never runs as root.
- [ ] Image writing, flush, and post-write verification work.
- [ ] Errors are actionable and never produce false success.
- [ ] Logs/diagnostics are useful and privacy-conscious.
- [ ] Automated unit and integration suites pass.
- [ ] Debian package builds successfully from a clean checkout.
- [ ] `.deb` installs, launches, upgrades (when applicable), and uninstalls cleanly in a clean test environment.
- [ ] Hardware validation status is honestly documented.
- [ ] GitHub Release includes package, checksum, release notes, and supported-system details.
- [ ] Final report lists exact commands, test results, artifacts, commit IDs, and any remaining limitations.

## 15. Final reporting format

At each checkpoint and at completion, report:

1. **STATUS:** BLOCKED / IN PROGRESS / PASS / FAIL.
2. **PHASE:** current phase and checkpoint.
3. **VERIFIED:** evidence-backed facts and tests actually run.
4. **CHANGES:** exact files added/modified and purpose.
5. **TESTS:** commands, environment, pass/fail counts, and logs/artifacts.
6. **RISKS/BLOCKERS:** unresolved issues, owner decisions, and unsupported hardware.
7. **NEXT ACTION:** the next bounded task.

At final completion, include:
- Repository commit/release tag.
- Exact `.deb` filename, architecture, size, and SHA-256.
- Clean-install test environment and result.
- Hardware test matrix and evidence.
- Known limitations.
- Confirmation that copyright, third-party attribution, and non-affiliation notices were reviewed.

Do not use “100% complete,” “fully functional,” “tested,” or “ready for deployment” unless the corresponding acceptance evidence exists. If a requirement cannot be met, stop at the relevant checkpoint, explain the evidence, and propose the smallest safe corrective action.

---

## 16. Authoritative references

- ChromiumOS user recovery tools: https://chromium.googlesource.com/chromiumos/user-recovery-tools/
- Google's recovery metadata: https://dl.google.com/dl/edgedl/chromeos/recovery/recovery.conf
- ChromiumOS recovery flow documentation: https://chromium.googlesource.com/chromiumos/platform/initramfs/+/master/recovery/README.md
- Android brand guidelines: https://developer.android.com/distribute/marketing-tools/brand-guidelines
- Android robot and trademark legal guidance: https://partnermarketinghub.withgoogle.com/brands/android/legal-and-trademarks/
- Debian Policy: https://www.debian.org/doc/debian-policy/
- Debian packaging guide: https://www.debian.org/doc/manuals/packaging-tutorial/packaging-tutorial.en.pdf

**Owner directive:** Build the complete product. Preserve evidence. Protect user disks. Credit the owner and every upstream creator correctly. Ship a real, tested Debian package—not a demo.
