---
name: Workflow Converter
description: Convert legacy Markdown files containing hub-workflow or workflow fenced blocks into standalone canonical .workflow.yaml files. Use when migrating workflows from Obsidian LLM Hub or other Markdown-based workflow stores.
---

```skill-capabilities
workflows:
  - path: workflows/convert.workflow.yaml
    description: Extract one hub-workflow block from a Markdown file and save it as pure YAML
    inputVariables:
      - source_path
      - output_path
```

# Workflow Converter

Convert a legacy Markdown workflow into the canonical LLM Hub Workspace format.

## Rules

- The source must be a Markdown file containing exactly one fenced `hub-workflow` or `workflow` block.
- The output path must end in `.workflow.yaml`, normally under `workflows/`.
- The output contains only YAML. Do not retain the Markdown heading, prose, or code fence.
- Never overwrite or delete the source Markdown file unless the user explicitly asks.
- If migrating an external Skill, also update its `SKILL.md` workflow capability path to the generated `.workflow.yaml` path.
- For multiple workflow blocks, convert them separately and give each output a distinct kebab-case name.

When the user provides only a source path, derive the output from the workflow name or source filename, for example `Legacy/Spanish Translator.md` becomes `workflows/spanish-translator.workflow.yaml`. Confirm the derived path before running when an existing file could be overwritten.
