# GemiHub Desktop

Use project-relative paths and the Desktop plugin API. Verify details against the current files under `/usr/local/pkg/gemihub-desktop`.

## Identity and compatibility

- Plugin runtime host ID: `gemihub-desktop`
- Current plugin runtime host version: `0.10.0`
- Legacy plugin host ID accepted for patches: `llm-hub-workspace`
- External Skill consumer ID/version: `gemihub-desktop` / `0.1.0`

Do not confuse the external Skill consumer version with `PLUGIN_HOST_VERSION`. Plugin manifests use the runtime host version for `minAppVersion`.

Desktop requires the latest release tag and `manifest.version` to match after removing an optional leading `v`. It validates semantic versions, permissions, patch paths, and preview integrity before installation.

## Permissions and capability mapping

Prefer current native names for Desktop-only plugins:

| Permission | API |
|---|---|
| `files` | `api.workspaceFiles` and `api.files` |
| `storage` | `api.storage` |
| `network` | `api.network` |
| `llm` | `api.llm` |

For shared manifests, Desktop maps `drive` to file capability and `gemini` to LLM capability. When LLM chat is available, `api.gemini` aliases `api.llm`. Although the manifest validator recognizes legacy or ecosystem permission strings such as `calendar`, `gmail`, and `sheets`, the current Desktop `createPluginAPI` does not implement those properties. Keep such features optional.

## Workspace files

Use `api.workspaceFiles` for the active Desktop Workspace:

```ts
api.workspaceFiles?.current()
api.workspaceFiles?.inventory()
api.workspaceFiles?.read(path)
api.workspaceFiles?.create(path, content)
api.workspaceFiles?.update(path, content)
api.workspaceFiles?.createDirectory(path)
api.workspaceFiles?.rename(oldPath, newPath)
api.workspaceFiles?.delete(path)
```

Use Workspace-relative forward-slash paths. Do not add `workspace://`, `files://`, an absolute Workspace directory, or a Web Drive ID. Scope belongs to the selected API surface, not the path string. Binary reads return data URLs for recognized binary extensions; binary writes accept `ArrayBuffer`.

Use `api.files` only for the separately selected external Files root. It has the same relative-path rule and additionally exposes `search()` and `tree()`. Never pass a path returned by one root API directly to the other unless the plugin deliberately copies content between roots.

For a Workspace-wide plugin, select the capability once and keep all reads and
writes on the same surface:

```ts
const files = api.workspaceFiles;
if (!files) throw new Error("This plugin requires Workspace files.");
const workspace = await files.current();
const inventory = await files.inventory();
```

Plugins that use Workspace identity, inventory, or configured-model selection
must declare `minAppVersion: "0.10.0"` or newer.

```ts
const unsubscribe = api.onActiveFileChanged(({ path, name }) => {
  // Both values can be null.
});
```

The active-file callback currently exposes a path without its root. Use it for display/context, but do not feed that path into a root API unless the plugin already knows the source root. The current string-only `api.selectFile(path)` has the same ambiguity; do not depend on it for cross-root navigation.

The component type permits `{ api, language?, filePath? }`, but the current `PluginHost` renders sidebar components with only `api` and `language`. Do not depend on a `filePath` prop. Offer a picker backed by `api.workspaceFiles.inventory()` when the view needs a Workspace file. If a picker combines Workspace and Files results, retain the chosen API/root alongside the relative path in plugin state.

## Network, LLM, storage, and assets

```ts
const response = await api.network?.request({
  url: "https://api.example.com/data",
  method: "GET",
  headers: { Accept: "application/json" },
});
// response: { status, headers, body, bodyBase64 }

const models = await api.llm?.listModels() ?? [];
// Populate a select with model.id as its value and model.label as its label.
const selectedModelId = models[0]?.id;
if (!selectedModelId) throw new Error("Configure an LLM model in Desktop settings.");
const text = await api.llm!.chat(messages, { modelId: selectedModelId, systemPrompt });
if (!text.trim()) throw new Error("The selected model returned an empty response.");
```

Use the exact opaque `id` returned by `listModels()` as `modelId`. Do not ask
users to type provider/model names, and do not reconstruct an ID from the
`provider` or `model` fields. Those fields are display metadata; `modelId`
selects the configured profile, including CLI providers such as Codex. Keep the
action disabled and direct the user to Desktop model settings when the list is
empty. The older `model` option is only a compatibility fallback.

Use `api.storage?.get`, `set`, and `getAll`. Use `api.assets.fetch(name)` for manifest-declared HTTPS assets. The backend blocks private/internal hosts, validates redirects, caches downloads, and limits asset size.

## UI registration

- `registerView`, `registerSettingsTab`, `registerSlashCommand`, and Web-compatible `registerWidget` exist.
- `registerWidget` accepts the shared `WidgetDef` contract from `shared.md`. Desktop renders `render(config, ctx)`, settings through `ConfigEditor`, and Open targets through `filePathOf` or `externalUrlOf`.
- Sidebar views are rendered by the current `PluginHost`.
- The type permits `location: "main"`, but the current host only selects sidebar views for plugin tabs. For a shared plugin, choose a sidebar fallback through a build define rather than assuming main-view rendering.

## Installation behavior

Desktop downloads required release files plus declared host patches, applies patches before atomic installation, protects `manifest.json` from patch modification, records applied patch hashes, and removes patch files from the installed plugin. Patch paths must be safe relative paths; GitHub release assets themselves are flat filenames.
