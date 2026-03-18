# Prototype Pipeline — Claude Code Agent System

A lean AI agent pipeline that transforms multi-format inputs (text briefs, Figma files, or
Ouroboros seeds) into fully-generated, interactive React/Vue SPA prototypes — for visual
and experience validation of workflows and designs.

Inspired by the design pipeline's conceptual process, but architecturally independent.
No backend required. No human code. Configurable byproduct outputs.

---

## Pipeline Map

```
╔══════════════════════════════════════════════════════════════════╗
║  INPUT FORMS                                                      ║
║                                                                   ║
║  [Text Brief]   [Figma File URL]   [Ouroboros Seed]               ║
║       │                │                  │                       ║
║       └────────────────┴──────────────────┘                       ║
║                        │                                          ║
║                        ▼                                          ║
║              pipeline-intake-agent                                ║
║              (pipeline.config.json)                               ║
╚════════════════════════╪═════════════════════════════════════════╝
                         │
                         ▼
              requirements-analyst
              (requirements.json)
                         │
                         ▼
                  screen-mapper
                 (screen-map.json)
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
   flow-diagram-builder    journey-diagram-builder
      (flow.mmd)               (journey.mmd)
              │                     │
              └──────────┬──────────┘
                         │
               [lo_fi_enabled?]
                    │       │
                   YES      NO
                    │       │
                    ▼       │
       lo-fi-wireframe-builder
       (lo-fi.excalidraw)
       ── GATE: review wireframe ──
                    │       │
                    └───────┘
                         │
                         ▼
                  spa-generator
                (spa/ file tree)
                         │
                         ▼
               prototype-validator
           (validation-report.json)
                         │
                         ▼
               delivery-sequencer
             (delivery-package.json
              PROTOTYPE.md)
```

---

## Agents

| Agent | Input | Output |
|---|---|---|
| `pipeline-intake-agent` | Brief / Figma URL / Seed path | `pipeline.config.json`, `pipeline-intake.json` |
| `requirements-analyst` | `pipeline.config.json` + input source | `requirements.json` |
| `screen-mapper` | `requirements.json` | `screen-map.json` |
| `flow-diagram-builder` | `screen-map.json`, `requirements.json` | `flow.mmd` |
| `journey-diagram-builder` | `requirements.json`, `screen-map.json` | `journey.mmd` |
| `lo-fi-wireframe-builder` | `screen-map.json`, `flow.mmd` | `lo-fi.excalidraw` (+ `lo-fi.svg` fallback) |
| `spa-generator` | `screen-map.json`, `requirements.json`, (`lo-fi.excalidraw`) | `spa/` file tree |
| `prototype-validator` | `spa/` | `validation-report.json` |
| `delivery-sequencer` | All artifacts | `delivery-package.json`, `PROTOTYPE.md` |
| `iteration-agent` | Existing run + change description | Updated artifacts, `iteration-plan.json`, iteration history |

---

## Stage Flow Detail

### Always-on byproducts
- **Flow diagram** (`flow.mmd`) — Mermaid flowchart of screen navigation
- **User journey** (`journey.mmd`) — Mermaid journey diagram per persona

### Optional byproduct
- **Lo-fi wireframe** (`lo-fi.excalidraw`) — Excalidraw JSON layout per screen
  - Enabled via `lo_fi_enabled: true` in `pipeline.config.json`
  - When enabled, **gates the SPA stage** — screens in the wireframe define routes and components

### SPA output
- React or Vue (configured per run)
- No backend — all state is simulated via local state + mock data
- Token-styled when design tokens are present as input (7 design system modes)
- Includes a **Pipeline Shell** — a fixed top bar with tabs to switch between the prototype and its byproducts (Flow, Journey, Wireframe, Artifacts) without leaving the browser
- All outputs written to `spa/` within the run directory

---

## Input Forms

The pipeline accepts any of:

| Input | How to provide |
|---|---|
| Text brief | Write to `brief.md` or `brief.txt` in the working directory |
| Figma file | Set `source.figma_file_url` in `pipeline.config.json` |
| Ouroboros seed | Set `source.seed_path` in `pipeline.config.json` |

Multiple input types may be combined. The intake agent merges them.

---

## Commands

```
proto                          — start from a text brief (creates a new run directory)
proto:seed <path/to/seed.yaml> — start from an Ouroboros seed YAML
proto:figma <figma-url>        — start from a Figma file URL
proto:iterate <change>         — apply a change to an existing run (re-runs only affected stages)
```

Type the command and Claude Code will run all pipeline stages end-to-end,
pausing at the lo-fi gate if wireframes are enabled.

---

## Iteration

`proto:iterate` applies a free-text change description to an existing run:

```
proto:iterate add a notifications screen accessible from the nav bar
proto:iterate replace the tab bar with a side navigation drawer
proto:iterate add a filter panel to the memory list screen
proto:iterate change the design system to Material UI
```

The iteration-agent classifies the change, determines the minimum set of pipeline stages to re-run, applies surgical edits (editing only affected files rather than regenerating everything), re-validates, and updates `PROTOTYPE.md` and `delivery-package.json`. Each iteration is recorded in the run's iteration history.

---

## Quick Start

### From brief
```
proto
```
Claude will ask for output directory, framework, lo-fi preference, and design system input,
then run all stages automatically.

### From Ouroboros seed
```
proto:seed /path/to/my.seed.yaml
```
Skips any interview questions already resolved by the seed.

### From Figma file
```
proto:figma https://www.figma.com/design/...
```
Intake agent extracts screen structure from the Figma file.

---

## Configuration

### `pipeline.config.json` schema

```json
{
  "project_name": "string",
  "run_id": "string (auto-generated)",
  "working_dir": "/absolute/path/to/run/dir",
  "source": {
    "mode": "brief | figma | seed | combined",
    "brief_path": "relative to working_dir (optional)",
    "figma_file_url": "https://figma.com/design/... (optional)",
    "seed_path": "absolute path to .yaml seed (optional)"
  },
  "output": {
    "output_dir": "/absolute/path/to/output (defaults to working_dir/<slug>-YYYYMMDD/)",
    "framework": "react | vue",
    "lo_fi_enabled": false,
    "lo_fi_gate": true
  },
  "design_system": {
    "mode": "none | css | json_tokens | figma | tailwind | named | description",
    "css_path": "path to .css file with custom properties (optional)",
    "json_tokens_path": "path to Style Dictionary / W3C / Figma tokens JSON (optional)",
    "figma_file_url": "Figma URL for token extraction via MCP (optional)",
    "tailwind_config_path": "path to tailwind.config.* (optional)",
    "named_system": "shadcn | antd | mui | chakra (optional)",
    "description": "plain-text styling intent, e.g. 'dark theme, Inter font, purple primary' (optional)"
  }
}
```

---

## Run Directory Layout

```
runs/
  <run_id>/
    pipeline.config.json
    pipeline-intake.json
    requirements.json
    screen-map.json
    flow.mmd
    journey.mmd
    lo-fi.excalidraw         (optional — when lo_fi_enabled: true)
    lo-fi-index.json         (optional — when lo_fi_enabled: true)
    spa-manifest.json
    iteration-plan.json      (optional — written on first proto:iterate)
    validation-report.json
    delivery-package.json
    PROTOTYPE.md
    spa/
      index.html
      vite.config.js
      package.json
      public/
        pipeline-manifest.json   (byproduct content for the shell)
      src/
        App.{jsx|vue}
        main.{jsx|js}
        router.{jsx|js}
        shell/
          PipelineShell.{jsx|vue}
          tabs/
            FlowTab.{jsx|vue}
            JourneyTab.{jsx|vue}
            WireframeTab.{jsx|vue}
            ArtifactsTab.{jsx|vue}
        views/
          <ScreenName>View.{jsx|vue}
        components/
          <ComponentName>.{jsx|vue}
        data/
          mock-data.json
        styles/
          index.css
          shell.css
          tokens.css           (optional — when design system provided)
```
