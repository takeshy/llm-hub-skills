# GemiHub Web

Use Drive IDs and the Web plugin runtime. Verify details against the current files under `/usr/local/pkg/gemihub`.

## Identity and installation

- The external Skill consumer ID is `gemihub`, version `0.1.0` in `app/services/external-skills.server.ts`.
- A plugin release requires `manifest.json` and `main.js`; `styles.css` is optional.
- The Web installer currently stores only `manifest.json`, `main.js`, and optional `styles.css` under the plugin Drive folder. It ignores Desktop patch assets.
- `minAppVersion` is required in the manifest parser but currently reserved rather than enforced.
- The Web installer reports the GitHub release tag separately and does not currently enforce equality with `manifest.version`. Enforce equality in cross-host projects because Desktop does.

## Permissions

Use only these Web permission names:

```text
gemini, drive, storage, calendar, gmail, sheets
```

Permission-gated properties are optional. Test for their presence.

## Files and active selection

Web centers operations on Google Drive `fileId` and `fileName`:

```ts
api.drive?.readFile(fileId)
api.drive?.searchFiles(query)
api.drive?.listFiles(folder?)
api.drive?.createFile(name, stringOrArrayBuffer) // -> { id, name }
api.drive?.updateFile(fileId, stringOrArrayBuffer)
api.drive?.rebuildTree()
```

Writes update the local-first cache and later sync to Drive. New files can receive temporary `new:*` IDs. Do not interpret a Drive ID as a filesystem path.

```ts
const unsubscribe = api.onActiveFileChanged(({ fileId, fileName, mimeType }) => {
  // Each value can be null.
});
api.selectFile(fileId, fileName, mimeType?);
```

View components receive `{ api, language?, fileId?, fileName? }`.

## LLM, storage, assets, and network

Use `api.gemini?.chat(messages, { model?, systemPrompt? })`. Web messages may include documented attachments. There is no general `api.network` surface; browser `fetch` runs with plugin privileges, while declared assets should use `api.assets.fetch(name)` so the server validates and caches them.

Use `api.storage?.get`, `set`, and `getAll` for plugin-scoped Drive storage. Use optional `calendar`, `gmail`, and `sheets` only after checking availability and entitlement.

## UI registration

- Use `registerView` for `sidebar` or `main`; main views can declare dot-prefixed `extensions`.
- Use `registerSettingsTab` for plugin settings.
- Use `registerWidget` for dashboard widget types.
- Do not use `registerSlashCommand`: the current Web `PluginAPI` type and `createPluginAPI` implementation do not expose it, even though the prose guide contains an older example.

## Local development and security

Place local builds under `/usr/local/pkg/gemihub/plugins/<id>/` when working in the Web source tree. Local plugins bypass normal permission gating. Plugins execute trusted browser code with DOM and fetch access; do not treat the runtime as a security sandbox.
