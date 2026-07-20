# llm-hub-skills

Reusable AI agent skills for LLM Hub ecosystem plugins.

Each skill lives under `skills/<skill-id>/` and includes a `manifest.json`
declaring its identity, version, description, and plugin compatibility constraints.

## `manifest.json`

Every skill MUST include a manifest at `skills/<skill-id>/manifest.json`.

Example:

```json
{
  "id": "okf",
  "name": "OKF Authoring",
  "version": "0.1.1",
  "description": "Author Open Knowledge Format (OKF) knowledge bundles in the vault so they can be used as LLM Hub ecosystem knowledge sources.",
  "hostPatches": {
    "gemihub": ["patches/gemihub.patch"]
  },
  "compatibility": {
    "plugins": [
      { "id": "gemini-helper", "minVersion": "1.16.0" },
      { "id": "llm-hub", "minVersion": "0.24.0" },
      { "id": "local-llm-hub", "minVersion": "0.16.0" },
      { "id": "gemihub", "minVersion": "0.1.0" }
    ]
  }
}
```

Fields:

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `id` | Yes | string | Stable skill identifier. MUST match the directory name in `skills/<skill-id>/`. Use lowercase kebab-case. |
| `name` | Yes | string | Human-readable skill name shown in plugin UIs and catalogs. |
| `version` | Yes | string | Skill version. Use semantic versioning (`MAJOR.MINOR.PATCH`). |
| `description` | Yes | string | One-sentence summary of what the skill helps an agent do. |
| `hostPatches` | No | object | Host-specific unified diff files to apply during import. Keys are plugin IDs; paths are relative to the skill folder. |
| `compatibility` | Yes | object | Compatibility declaration for host plugins. |
| `compatibility.plugins` | Yes | array | List of plugins that can consume this skill. At least one plugin entry is required. |
| `compatibility.plugins[].id` | Yes | string | Stable plugin identifier, such as `gemini-helper`. |
| `compatibility.plugins[].minVersion` | Yes | string | Minimum plugin version that supports this skill. Use semantic versioning. |

Rules:

- `manifest.json` MUST be valid JSON. Comments and trailing commas are not allowed.
- `id` is immutable after publication. Create a new skill directory when a rename would break consumers.
- `version` MUST change whenever the skill content or manifest changes.
- A skill can support multiple plugins by adding entries to `compatibility.plugins`.
- If a skill needs host-specific instructions, list patch files in `hostPatches` instead of forking the whole skill.
- Additional fields are allowed, but consumers MUST ignore unknown fields.

## Skills

- **build-gemihub-plugins** — Create, port, validate, and release plugins for GemiHub Web and GemiHub Desktop.
- **okf** — Author and maintain Open Knowledge Format (OKF) knowledge bundles in the vault, for use as LLM Hub ecosystem knowledge sources.
- **workflow-converter** — Convert Markdown `hub-workflow` blocks into canonical standalone `.workflow.yaml` files for LLM Hub Workspace.
