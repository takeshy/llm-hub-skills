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
| `compatibility` | Yes | object | Compatibility declaration for host plugins. |
| `compatibility.plugins` | Yes | array | List of plugins that can consume this skill. At least one plugin entry is required. |
| `compatibility.plugins[].id` | Yes | string | Stable plugin identifier, such as `gemini-helper`. |
| `compatibility.plugins[].minVersion` | Yes | string | Minimum plugin version that supports this skill. Use semantic versioning. |

Rules:

- `manifest.json` MUST be valid JSON. Comments and trailing commas are not allowed.
- `id` is immutable after publication. Create a new skill directory when a rename would break consumers.
- `version` MUST change whenever the skill content or manifest changes.
- A skill can support multiple plugins by adding entries to `compatibility.plugins`.
- Additional fields are allowed, but consumers MUST ignore unknown fields.

## Skills

- **okf** — Author and maintain Open Knowledge Format (OKF) knowledge bundles in the vault, for use as LLM Hub ecosystem knowledge sources.
