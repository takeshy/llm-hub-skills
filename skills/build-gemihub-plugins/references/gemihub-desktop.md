# GemiHub Desktop

Use project-relative paths and the Desktop plugin API. Verify details against the current files under `/usr/local/pkg/gemihub-desktop`.

## Identity and compatibility

- Plugin runtime host ID: `gemihub-desktop`
- Current plugin runtime host version: `0.9.0`
- Legacy plugin host ID accepted for patches: `llm-hub-workspace`
- External Skill consumer ID/version: `gemihub-desktop` / `0.1.0`

Do not confuse the external Skill consumer version with `PLUGIN_HOST_VERSION`. Plugin manifests use the runtime host version for `minAppVersion`.

Desktop requires the latest release tag and `manifest.version` to match after removing an optional leading `v`. It validates semantic versions, permissions, patch paths, and preview integrity before installation.

## Permissions and capability mapping

Prefer current native names for Desktop-only plugins:

| Permission | API |
|---|---|
| `files` | `api.files` and `api.projectFiles` |
| `storage` | `api.storage` |
| `network` | `api.network` |
| `llm` | `api.llm` |

For shared manifests, Desktop maps `drive` to file capability and `gemini` to LLM capability. When LLM chat is available, `api.gemini` aliases `api.llm`. Although the manifest validator recognizes legacy or ecosystem permission strings such as `calendar`, `gmail`, and `sheets`, the current Desktop `createPluginAPI` does not implement those properties. Keep such features optional.

## Workspace and active-project files

`api.files` addresses the Desktop workspace root:

```ts
api.files?.read(path)
api.files?.search(query, limit?)
api.files?.tree()
api.files?.create(path, content)
api.files?.update(path, content)
api.files?.createDirectory(path)
api.files?.rename(oldPath, newPath)
api.files?.delete(path)
```

`api.projectFiles` addresses every file in the active project:

```ts
api.projectFiles?.current()
api.projectFiles?.inventory()
api.projectFiles?.read(path)
api.projectFiles?.create(path, content)
api.projectFiles?.update(path, content)
api.projectFiles?.createDirectory(path)
api.projectFiles?.rename(oldPath, newPath)
api.projectFiles?.delete(path)
```

Use project-relative forward-slash paths. Binary reads return data URLs for recognized binary extensions; binary writes accept `ArrayBuffer`. Do not pass a Web Drive ID into these methods.

```ts
const unsubscribe = api.onActiveFileChanged(({ path, name }) => {
  // Both values can be null.
});
api.selectFile(path);
```

The component type permits `{ api, language?, filePath? }`, but the current `PluginHost` renders sidebar components with only `api` and `language`. Do not depend on a `filePath` prop. Subscribe with `onActiveFileChanged` or offer a picker backed by `projectFiles.inventory()` when the view needs a file.

## Network, LLM, storage, and assets

```ts
const response = await api.network?.request({
  url: "https://api.example.com/data",
  method: "GET",
  headers: { Accept: "application/json" },
});
// response: { status, headers, body, bodyBase64 }

const text = await api.llm?.chat(messages, { model, systemPrompt });
```

Use `api.storage?.get`, `set`, and `getAll`. Use `api.assets.fetch(name)` for manifest-declared HTTPS assets. The backend blocks private/internal hosts, validates redirects, caches downloads, and limits asset size.

## UI registration

- `registerView`, `registerSettingsTab`, `registerSlashCommand`, and Web-compatible `registerWidget` exist.
- `registerWidget` accepts the shared `WidgetDef` contract from `shared.md`. Desktop renders `render(config, ctx)`, settings through `ConfigEditor`, and Open targets through `filePathOf` or `externalUrlOf`.
- Sidebar views are rendered by the current `PluginHost`.
- The type permits `location: "main"`, but the current host only selects sidebar views for plugin tabs. For a shared plugin, choose a sidebar fallback through a build define rather than assuming main-view rendering.

## Installation behavior

Desktop downloads required release files plus declared host patches, applies patches before atomic installation, protects `manifest.json` from patch modification, records applied patch hashes, and removes patch files from the installed plugin. Patch paths must be safe relative paths; GitHub release assets themselves are flat filenames.
