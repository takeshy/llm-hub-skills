# One release for GemiHub Web and Desktop

Use this pattern only after reading the shared and both host references.

## Design the adapter boundary

Keep business logic and React components shared. Put identifier and capability translation in `src/host.ts`. Adapt only the methods the plugin actually calls.

```ts
declare const __GEMIHUB_DESKTOP__: boolean;

interface DesktopFiles {
  read(path: string): Promise<string>;
  create(path: string, content: string | ArrayBuffer): Promise<void>;
  update(path: string, content: string | ArrayBuffer): Promise<void>;
}

interface DesktopAPI {
  projectFiles?: DesktopFiles;
  [key: string]: unknown;
}

export function adaptPluginAPI<T>(input: T): T {
  if (!__GEMIHUB_DESKTOP__) return input;
  const api = input as T & DesktopAPI;
  const files = api.projectFiles;
  if (!files) throw new Error("This plugin requires GemiHub Desktop 0.8.1 or newer.");
  return Object.assign(api, {
    drive: {
      readFile(path: string) { return files.read(path); },
      async createFile(name: string, content: string | ArrayBuffer) {
        await files.create(name, content);
        return { id: name, name };
      },
      updateFile(path: string, content: string | ArrayBuffer) {
        return files.update(path, content);
      },
    },
  });
}

export const mainLocation: "sidebar" | "main" =
  __GEMIHUB_DESKTOP__ ? "sidebar" : "main";
```

Call `adaptPluginAPI(hostAPI)` once in `onload`. Do not implement `searchFiles`, `listFiles`, or ID-to-path translation unless the plugin needs them and their semantics are defined. For binary Desktop reads, decode the data URL returned by project file reading before passing bytes to a decoder.

Do not port a Web main view by expecting Desktop to pass Web-style `fileId`/`fileName` or even the optional typed `filePath` prop. The current Desktop sidebar renderer passes only `api` and `language`. Subscribe to `onActiveFileChanged`, or add a project-relative picker using `projectFiles.inventory()`, and keep that selection boundary explicit.

## Generate a release-bundle patch

Build Web as canonical `main.js`, build a Desktop variant from the same entrypoint, and diff the bundles:

```js
import esbuild from "esbuild";
import { mkdir, rm, writeFile } from "node:fs/promises";
import { spawnSync } from "node:child_process";

const common = {
  entryPoints: ["src/main.ts"],
  bundle: true,
  platform: "browser",
  format: "cjs",
  target: "es2020",
  external: ["react", "react-dom", "react-dom/client"],
  loader: { ".ts": "ts", ".tsx": "tsx" },
};

await mkdir(".build", { recursive: true });
await esbuild.build({
  ...common,
  define: { __GEMIHUB_DESKTOP__: "false" },
  outfile: "main.js",
});
await esbuild.build({
  ...common,
  define: { __GEMIHUB_DESKTOP__: "true" },
  outfile: ".build/main.gemihub-desktop.js",
});

const diff = spawnSync("diff", [
  "-u", "--label", "a/main.js", "--label", "b/main.js",
  "main.js", ".build/main.gemihub-desktop.js",
], { encoding: "utf8" });
if (diff.status !== 1 || !diff.stdout.startsWith("--- a/main.js\n+++ b/main.js\n")) {
  throw new Error(diff.stderr || "Could not generate Desktop patch");
}
await mkdir("patches", { recursive: true });
await writeFile("patches/gemihub-desktop.patch", diff.stdout);
await rm(".build", { recursive: true, force: true });
```

Keep both builds unminified when a one-line minified bundle would turn a two-branch change into a full-bundle patch. Review the patch; its targets must be `a/main.js` and `b/main.js`. This is a patch to the plugin release bundle, not to GemiHub Desktop source files.

## Declare cross-host compatibility

```json
{
  "id": "example-plugin",
  "name": "Example Plugin",
  "version": "0.1.0",
  "minAppVersion": "0.8.1",
  "description": "Example shared plugin.",
  "author": "Your Name",
  "permissions": ["storage", "drive", "gemini"],
  "assets": [],
  "hostPatches": {
    "gemihub-desktop": ["patches/gemihub-desktop.patch"]
  }
}
```

Use shared permission aliases only when the adapter needs them: Desktop interprets `drive` as file access and `gemini` as LLM access. Add `network` only for Desktop requests; Web has no matching permission/API. Keep Web-only premium APIs optional.

Do not branch or patch `registerWidget`. Web and Desktop 0.9.0 share the same `WidgetDef` contract and persisted widget type. Set `minAppVersion` to at least `0.9.0` when the plugin registers a dashboard widget; plugins that do not use this API can retain their lower compatible version.

## Publish all assets

Include these paths in build artifacts and releases:

```yaml
files: |
  main.js
  manifest.json
  styles.css # omit when the plugin has no stylesheet
  patches/gemihub-desktop.patch
```

GitHub publishes the patch asset using its basename. The Desktop manager resolves the manifest's safe logical path to that basename. GemiHub Web ignores the Desktop `hostPatches` declaration and installs its normal `main.js`.

## Verify the result

1. Run a clean install, type-check, both builds, and all tests.
2. Assert the generated patch is small and contains only expected build-selected differences.
3. Apply it with Desktop's `applyHostPatches` implementation or an equivalent unified-diff test and inspect patched `main.js`.
4. Test Web Drive IDs separately from Desktop relative paths.
5. Test binary reads and writes when supported extensions are involved.
6. Validate external asset sizes against the Desktop backend limit and verify HTTPS/private-host rules.
7. Confirm `manifest.version`, `package.json` version, lockfile root version, and release tag agree.
8. Confirm `manifest.json`, `main.js`, optional `styles.css`, and the patch basename are attached to the release.
9. Publish the draft only after both hosts pass installation smoke tests.

## Current examples

Inspect these repositories rather than copying blindly:

- `/usr/local/pkg/hub-audio-score` — binary file adaptation, assets, and main-view fallback
- `/usr/local/pkg/hub-accounting` — project inventory/list adaptation and selection
- `/usr/local/pkg/hub-ronginus` — minimal create-file adapter
