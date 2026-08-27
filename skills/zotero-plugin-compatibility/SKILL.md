---
name: zotero-plugin-compatibility
description: Diagnose and update Zotero plugins across major-version upgrades, including manifest compatibility, API breakage, reader UI integration, builds, and XPI verification. Use for plugin maintenance or upgrade failures; not for ordinary Zotero usage or translation-quality questions.
---

# Zotero Plugin Compatibility

Use this skill when a Zotero plugin is rejected, disabled, broken, or missing an expected entry point after a Zotero upgrade. The goal is a reproducible, reviewable compatibility fix with evidence—not merely a package that builds.

## Start with the failure boundary

Classify the symptom before editing code:

- **Install/activation failure:** Zotero refuses the XPI or disables it. Inspect the packaged `manifest.json`, especially `applications.zotero.strict_min_version` and `strict_max_version`.
- **Startup/runtime failure:** the plugin installs but throws during startup or when a command runs. Trace the failing module and compare the API against the target Zotero developer documentation.
- **Entry-point failure:** an action is available in the item/library view but not in the PDF reader. Treat these as different extension surfaces; investigate reader-specific APIs and toolbar/menu attachment points rather than changing only the item-list command.
- **Build/distribution failure:** TypeScript, bundling, packaging, update metadata, or the generated XPI is wrong. Verify the artifact, not only the source tree.

Keep these causes separate in the report. A manifest range can explain installation rejection, but it does not prove that runtime APIs are compatible.

## Establish the repository state safely

1. Record the repository, remote, current branch, target Zotero version, plugin version, and reproduction steps.
2. Inspect `git status` before changing anything. Preserve unrelated user modifications; use a new branch when a repository workflow permits it.
3. Read the build scripts, `addon/manifest.json`, bootstrap/startup code, localization files, and the modules involved in the reported feature. Follow actual call paths instead of relying on the plugin name or README.
4. If an upstream issue or pull request exists, inspect its files and diff. Treat it as evidence and a candidate patch, not as automatically correct; check that its changes address the reported target version and do not introduce unrelated behavior.

## Research the target version

For a major-version migration, consult current primary sources first:

- Zotero's developer documentation and version changelog for compatibility metadata and breaking API changes.
- The plugin's upstream issues, pull requests, releases, and update manifest.
- Relevant source/API definitions in the checked-out project.

Record the exact claim each source supports. Do not generalize from one broken plugin to all plugins. If the official migration page is unavailable or unclear, say what was verified from source and what remains uncertain.

## Apply the smallest justified compatibility patch

Prefer a narrow patch that is tied to the observed failure:

- Update `strict_max_version` only after confirming the target major version is intended and the code has been checked for API compatibility.
- If an upstream patch is relevant, apply or reimplement only the needed hunks and preserve local changes. Review every changed file.
- For metadata/API extraction features, support the fields the target Zotero actually populates (for example DOI, URL, archive ID, and `extra`) with an explicit precedence order and clear failure messages. Keep parsing functions small and testable.
- Do not loosen compatibility ranges as a substitute for fixing removed, renamed, or changed APIs.
- Do not modify proxy settings, expose local services, or bypass security controls just to make a build work.

When adding a reader entry point, first map the lifecycle: reader opened → toolbar/menu registered → action receives the reader/item context → translation/download task runs → attachment is created and opened. Confirm cleanup on reader/window close and avoid duplicate registrations.

## Build and verify the actual XPI

Use the repository's documented package manager and scripts. Typical checks are:

```bash
npm install
npm run build
```

If the command reaches a local proxy such as `127.0.0.1:<port>` and fails with `EPERM`, distinguish sandbox/network isolation from a project or proxy error. Preserve the user's `HTTP_PROXY`, `HTTPS_PROXY`, `ALL_PROXY`, and related Node proxy settings; validate connectivity from the host-approved environment instead of silently unsetting them or changing a proxy listener to `0.0.0.0`.

After building, inspect the generated XPI rather than trusting build output:

```bash
unzip -p path/to/plugin.xpi manifest.json
```

Verify the packaged plugin ID, version, target Zotero range, update URL, and any newly bundled files. Also run the project's typecheck/tests when available. A successful bundle is necessary but does not prove Zotero runtime behavior.

## Manual verification plan

Provide a short test matrix tailored to the change:

- install the generated XPI in the target Zotero major version;
- restart Zotero if the plugin manager or bootstrap requires it;
- exercise the original failing path;
- exercise every newly supported identifier/input form;
- for reader UI work, open a real PDF reader, confirm the control appears once, invoke it, and verify the resulting attachment is created under the expected item and opens correctly;
- inspect the Zotero error console for startup, Reader, toolbar, attachment, and network errors.

State what was actually tested versus what still requires a user's Zotero GUI or account. Do not claim full compatibility from a manifest inspection or a successful build alone.

## Deliverable format

End with:

1. root cause and confidence level;
2. files and behavior changed;
3. upstream evidence used, including issue/PR numbers when applicable;
4. commands and artifact-level verification results;
5. exact XPI path and installation steps;
6. remaining manual tests, limitations, and rollback/upgrade advice.

Do not commit, push, publish a release, or alter the user's global Zotero configuration unless the user explicitly asks for that action.
