# Excalidraw Plugin for Claude Code

A Claude Code plugin that generates architecture diagrams as Excalidraw files from natural language descriptions. Produces `.excalidraw`, `.svg`, and `.png` outputs with automatic layout via dagre.

![Example Output](https://raw.githubusercontent.com/yogsma/excalidraw-plugin/main/examples/web-app-architecture.png)

## Installation

Add to your Claude Code settings (`~/.claude/settings.json`):

```json
{
  "plugins": ["https://github.com/yogsma/excalidraw-plugin"]
}
```

Dependencies (`@dagrejs/dagre`, `sharp`) install automatically on first use.

## Usage

### `/draw` Command

```
/draw React frontend connecting to an Express API with PostgreSQL and Redis cache
```

This generates three files in your working directory:
- `web-application.excalidraw` — editable in [excalidraw.com](https://excalidraw.com)
- `web-application.svg` — scalable vector graphic
- `web-application.png` — 2x resolution raster image

### Diagram-from-Code Agent

The plugin includes an autonomous agent that analyzes your codebase and generates an architecture diagram automatically. It discovers services, databases, queues, and their connections by examining:

- Package manifests (`package.json`, `go.mod`, etc.)
- Docker configurations
- Database connection strings and ORM configs
- HTTP clients and route definitions
- Message queue clients

## How It Works

```
Natural Language → Claude → IR JSON → dagre layout → .excalidraw → SVG + PNG
```

1. Claude parses your description into an intermediate representation (IR) JSON
2. `generate.mjs` builds a dagre graph, runs layout, and converts to Excalidraw elements
3. `export.mjs` renders the Excalidraw file to SVG and rasterizes to PNG via sharp

## IR Format

The intermediate representation that Claude generates:

```json
{
  "title": "System Architecture",
  "direction": "TB",
  "nodes": [
    { "id": "frontend", "label": "Frontend\n(React)", "type": "component", "color": "blue" },
    { "id": "api", "label": "API Server\n(Express)", "type": "component", "color": "green" },
    { "id": "db", "label": "PostgreSQL", "type": "database", "color": "orange" }
  ],
  "edges": [
    { "from": "frontend", "to": "api", "label": "REST" },
    { "from": "api", "to": "db", "label": "SQL" }
  ],
  "groups": [
    { "id": "backend", "label": "Backend", "members": ["api", "db"] }
  ]
}
```

## Color Palette

| Color  | Background | Stroke   | Use Case           |
|--------|-----------|----------|--------------------|
| blue   | #a5d8ff   | #1971c2  | Frontend, UI       |
| green  | #b2f2bb   | #2f9e44  | APIs, Services     |
| orange | #ffd8a8   | #e8590c  | Databases, Storage |
| red    | #ffc9c9   | #e03131  | Auth, Security     |
| purple | #d0bfff   | #7048e8  | Infrastructure     |
| yellow | #ffec99   | #f08c00  | Queues, Events     |
| gray   | #dee2e6   | #495057  | External / 3rd party |

## Node Types

- **component** — rounded rectangle (general services, apps)
- **database** — databases, data stores, caches
- **queue** — message queues, event buses
- **user** — users, actors, clients
- **cloud** — cloud services, external APIs

## Layout Directions

- `TB` — top-to-bottom (default, best for hierarchical architectures)
- `LR` — left-to-right (best for data pipelines and sequences)

## Plugin Structure

```
├── .claude-plugin/plugin.json          # Plugin manifest
├── commands/draw.md                    # /draw slash command
├── agents/diagram-from-code.md         # Codebase analysis agent
└── skills/excalidraw/
    ├── SKILL.md                        # Auto-triggering skill
    ├── references/
    │   ├── excalidraw-schema.md        # Excalidraw v2 format reference
    │   └── element-catalog.md          # Node types & color palette
    └── scripts/
        ├── package.json                # Dependencies
        ├── generate.mjs                # IR → Excalidraw JSON (dagre layout)
        └── export.mjs                  # Excalidraw → SVG + PNG
```

## Limitations

- SVG export uses clean geometric shapes (not hand-drawn style) — open the `.excalidraw` file in Excalidraw for the sketchy look
- Text width is estimated by character count, not font metrics
- Large diagrams (20+ nodes) may be cluttered — use groups to manage complexity
- `sharp` requires native compilation; falls back to SVG-only if unavailable

## License

MIT
