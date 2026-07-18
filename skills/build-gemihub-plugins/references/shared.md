# Shared GemiHub plugin contract

Use this reference for every plugin target. Then load the selected host reference.

## Primary sources

- Web guide: `/usr/local/pkg/gemihub/docs/integrations/plugins.md`
- Web types and runtime: `/usr/local/pkg/gemihub/app/types/plugin.ts`, `/usr/local/pkg/gemihub/app/services/plugin-api.ts`, `/usr/local/pkg/gemihub/app/services/plugin-loader.ts`
- Desktop guide: `/usr/local/pkg/gemihub-desktop/docs/okf/features/plugins.md`
- Desktop types and runtime: `/usr/local/pkg/gemihub-desktop/src/plugins/types.ts`, `api.ts`, and `manager.ts`

Re-read these files when present. They can change after this Skill is released.

## Release layout

Use a source repository such as:

```text
manifest.json
package.json
esbuild.config.mjs
src/main.ts
src/ui/Panel.tsx
styles.css                 # optional
patches/                   # cross-host only
  gemihub-desktop.patch
```

Attach `manifest.json` and CommonJS `main.js` to the latest GitHub Release. Attach `styles.css` when used. Attach every declared Desktop patch by its basename; the Desktop installer maps a safe logical path such as `patches/gemihub-desktop.patch` to the flat release asset.

## Entrypoint and lifecycle

```ts
class ExamplePlugin {
  onload(api: PluginAPI): void {
    api.registerView({
      id: "example",
      name: "Example",
      location: "sidebar",
      component: ExamplePanel,
    });
  }

  onunload(): void {
    // Unsubscribe events and stop timers, workers, or media here.
  }
}

module.exports = ExamplePlugin;
// module.exports.default = ExamplePlugin is also accepted.
```

Bundle for the browser as `format: "cjs"`. Externalize `react`, `react-dom`, and `react-dom/client`. A component receives `api` plus host-specific active-file props. Prefer `api.React` and `api.ReactDOM`, or external React imports resolved by the host shim; never bundle a second React copy.

## Registration support matrix

| Capability | Web | Desktop | Shared guidance |
|---|---|---|---|
| `registerView` | Yes | Yes | Sidebar views are portable. Check main-view rendering separately. |
| `registerSettingsTab` | Yes | Yes | The component receives `api` and optional `onClose`. |
| slash command | Not exposed by current Web types/runtime | `registerSlashCommand` | Do not call it in a shared unguarded path. The Web prose guide is stale here. |
| dashboard widget | `registerWidget(WidgetDef)` | `registerWidget(WidgetDef)` since 0.9.0 | Use one definition on both hosts. |
| `onunload` | Yes | Yes | Clean up resources owned by the plugin. |

Treat slash commands as part of the plugin design contract, but implement them only on a host that currently exposes registration.

## Portable dashboard widget

Use the Web `WidgetDef` contract unchanged on Desktop 0.9.0 or newer:

```tsx
api.registerWidget({
  type: "summary",
  label: "Summary",
  defaultConfig: { path: "" },
  defaultSize: { w: 6, h: 4 },
  render(config, ctx) {
    const value = config as { path?: string };
    return <SummaryWidget path={value.path ?? ""} onChange={(path) => ctx.onConfigChange?.({ ...value, path })} />;
  },
  ConfigEditor({ config, onChange }) {
    const value = config as { path?: string };
    return <input value={value.path ?? ""} onChange={(event) => onChange({ ...value, path: event.target.value })} />;
  },
  filePathOf(config) {
    const path = (config as { path?: string }).path?.trim();
    return path || undefined;
  },
});
```

The shared fields are `type`, `label`, `hiddenFromPalette?`, `icon?`, `defaultConfig`, `render(config, ctx)`, `defaultSize?`, `ConfigEditor?`, `filePathOf?`, and `externalUrlOf?`. The shared context fields are `host`, `size`, `widgetId?`, `dashboardFileId?`, `dashboardFileName?`, and `onConfigChange?`.

Keep the public `type` host-neutral. Desktop 0.9.0 stores exactly the same type as Web. Do not prefix it with a plugin or host ID.

## Manifest baseline

```json
{
  "id": "example-plugin",
  "name": "Example Plugin",
  "version": "0.1.0",
  "minAppVersion": "1.0.0",
  "description": "Describe the user-facing capability.",
  "author": "Your Name",
  "permissions": ["storage"],
  "assets": []
}
```

Use a stable ID containing only letters, numbers, dots, underscores, or hyphens. Use semantic versions. Web currently treats `minAppVersion` as required but reserved. For a cross-host release, replace it with the minimum Desktop plugin host version and ensure the release tag represents the same version as `manifest.version`; Desktop accepts an optional leading `v` on the tag.

Declare external assets as flat names with HTTPS URLs. Add SHA-256 when available. Fetch them through `api.assets.fetch(name)`.

## Baseline build

```js
import esbuild from "esbuild";

await esbuild.build({
  entryPoints: ["src/main.ts"],
  bundle: true,
  platform: "browser",
  format: "cjs",
  target: "es2020",
  outfile: "main.js",
  external: ["react", "react-dom", "react-dom/client"],
  loader: { ".ts": "ts", ".tsx": "tsx" },
});
```

## Validation checklist

- Parse `manifest.json` as strict JSON; reject comments and trailing commas.
- Confirm the ID, version, minimum host version, permissions, assets, and patch paths.
- Type-check and build from a clean dependency install.
- Run existing tests and a focused smoke test for changed runtime behavior.
- Confirm `main.js` exports a constructible plugin class.
- Confirm required release assets exist and patch basenames are unique.
- Confirm the current version tag does not already exist before relying on an automated release workflow.
- Run `git diff --check` and preserve unrelated worktree changes.
