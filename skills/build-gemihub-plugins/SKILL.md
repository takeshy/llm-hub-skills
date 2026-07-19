---
name: build-gemihub-plugins
description: Create new GemiHub plugins, port existing plugins, and make one GitHub Release support GemiHub Web and GemiHub Desktop. Use when implementing plugin entrypoints, React views, settings, commands, widgets, host adapters, manifest permissions, esbuild output, hostPatches, release assets, versioning, validation, or publication.
---

# Build GemiHub Plugins

Build against current host contracts instead of inferred APIs.

## Choose the target first

Classify the request before editing:

1. **GemiHub Web only** — read [references/shared.md](references/shared.md) and [references/gemihub-web.md](references/gemihub-web.md).
2. **GemiHub Desktop only** — read [references/shared.md](references/shared.md) and [references/gemihub-desktop.md](references/gemihub-desktop.md).
3. **One GitHub Release for both hosts** — read all references, including [references/cross-host.md](references/cross-host.md).

If the target is ambiguous, inspect the repository and ask only when the choice materially changes storage, UI placement, or release compatibility.

## Follow the workflow

1. Inspect the target repository, dirty worktree, package version, tags, build scripts, manifest, API calls, direct host URLs, registered views, and cleanup behavior. Preserve unrelated changes.
2. Verify every API against the current primary sources listed in the selected references. Treat implementation and type definitions as authoritative when prose documentation differs.
3. Inventory required capabilities before coding: Workspace files, storage, network, configured LLM models, active-file events, assets, views, settings, commands, and widgets.
4. Declare only the permissions actually used. Guard optional APIs before access.
5. Keep the CommonJS entrypoint and React contract shared. Add a host adapter only where identifiers or capability shapes differ.
6. For cross-host work, generate the Desktop unified diff from two builds of the same source. Apply the diff to the release `main.js`; never describe it as a patch to Desktop application source.
7. Use `hostPatches` only for differences that ordinary API adaptation cannot absorb, such as build-selected adapter activation, unsupported main-view placement, or a host-specific binary read path.
8. Build both variants, apply the generated patch with the host patch engine when available, run existing tests, validate manifest JSON, compare manifest version with the release tag, and run `git diff --check`.
9. Bump the plugin version before publication when the current tag already exists. Ensure the latest published GitHub Release contains every required asset.

## Keep the contract honest

- Externalize `react`, `react-dom`, and `react-dom/client`; use the host-provided React runtime.
- Export a class through `module.exports` or `module.exports.default`, implement `onload(api)`, and release listeners, timers, subscriptions, workers, and object URLs from `onunload()`.
- Treat view, settings, command, and widget registration as one design vocabulary, but consult the support matrix before calling a method. Use the same `registerWidget` definition on Web and Desktop 0.9.0 or newer; do not generate a host patch for widget registration.
- Use Drive IDs on Web and project-relative paths on Desktop. Do not silently pass one identifier form to the other.
- Use `api.files` for Desktop Workspace access. Populate Desktop model selectors from `api.llm.listModels()` and pass the exact selected ID as `modelId`.
- Do not claim Calendar, Gmail, Sheets, slash-command, or main-view support merely because a permission string or type alias exists elsewhere.
- Keep generated patches small and reviewable. Avoid minifying the two inputs when line-oriented diff stability matters.

## Hand off publication state

Report changed repositories, versions, generated release assets, test results, unsupported optional features, and whether changes are committed, pushed, drafted, or published. Never publish a draft or push a repository unless the user authorizes it.
