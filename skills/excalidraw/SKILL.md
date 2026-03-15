---
name: excalidraw
description: >
  Generates Excalidraw architecture diagrams from natural language.
  Triggers on: "draw a diagram", "architecture diagram", "visualize the system",
  "excalidraw", "system diagram", "generate diagram", "box and arrow diagram",
  "component diagram", "service map"
---

# Excalidraw Diagram Generation

You can generate architecture diagrams as Excalidraw files with SVG/PNG exports.

## When to Use

Use this skill when the user asks you to:
- Draw or create an architecture/system/component diagram
- Visualize a system, codebase, or service topology
- Generate an Excalidraw file
- Create a box-and-arrow diagram

## How It Works

1. Convert the user's description into an **IR (Intermediate Representation) JSON**
2. Run the generation script to produce an `.excalidraw` file with dagre auto-layout
3. Run the export script to produce `.svg` and `.png` files

## Setup

```bash
cd ${CLAUDE_PLUGIN_ROOT}/skills/excalidraw/scripts && if [ ! -d node_modules ]; then npm install; fi
```

## IR JSON Format

```json
{
  "title": "Diagram Title",
  "direction": "TB",
  "nodes": [
    { "id": "unique_id", "label": "Display Name\n(detail)", "type": "component", "color": "blue" }
  ],
  "edges": [
    { "from": "source_id", "to": "target_id", "label": "optional label" }
  ],
  "groups": [
    { "id": "group_id", "label": "Group Name", "members": ["node_id1", "node_id2"] }
  ]
}
```

## Color Palette

| IR Color | Background | Stroke   | Use Case          |
|----------|-----------|----------|-------------------|
| blue     | #a5d8ff   | #1971c2  | Frontend, UI      |
| green    | #b2f2bb   | #2f9e44  | APIs, Services    |
| orange   | #ffd8a8   | #e8590c  | Databases         |
| red      | #ffc9c9   | #e03131  | Auth, Security    |
| purple   | #d0bfff   | #7048e8  | Infrastructure    |
| yellow   | #ffec99   | #f08c00  | Queues, Events    |
| gray     | #dee2e6   | #495057  | External/3rd party|

## Node Types
- `component` — rounded rectangle (general services, apps)
- `database` — databases, storage systems
- `queue` — message queues, event buses
- `user` — users, actors
- `cloud` — cloud services, external APIs
- `group` — logical grouping container

## Scripts

**Generate:** `cat ir.json | node ${CLAUDE_PLUGIN_ROOT}/skills/excalidraw/scripts/generate.mjs output.excalidraw`

**Export:** `node ${CLAUDE_PLUGIN_ROOT}/skills/excalidraw/scripts/export.mjs output.excalidraw`

## Guidelines

- Use `TB` direction for hierarchical architectures, `LR` for pipelines
- Add `\n` in labels for multi-line text: `"API Gateway\n(Express)"`
- Group related services with the `groups` array
- Choose colors semantically (blue=frontend, green=API, orange=DB, etc.)
- Keep node count under 20 for readable diagrams; use groups for complexity
- Write IR to a temp file rather than piping inline JSON to avoid escaping issues
