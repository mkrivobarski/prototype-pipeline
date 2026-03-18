# Pipeline Schemas

Canonical JSON schemas for all pipeline artifacts. Agent files reference this document instead of repeating schema definitions. Each agent specifies only its own output fields.

---

## `pipeline-manifest.json`

Written by `shell-scaffolder`. Updated incrementally by every downstream agent via **read-merge-write** (never overwrite from scratch).

```json
{
  "project_name": "string",
  "run_id": "string",
  "framework": "react | vue",
  "generated_at": "ISO8601",
  "pipeline": {
    "requirements":  { "status": "pending | in_progress | ready", "updated_at": "ISO8601 | null" },
    "screen_map":    { "status": "pending | in_progress | ready", "updated_at": "ISO8601 | null" },
    "flow":          { "status": "pending | in_progress | ready", "updated_at": "ISO8601 | null" },
    "journey":       { "status": "pending | in_progress | ready", "updated_at": "ISO8601 | null" },
    "lo_fi":         { "status": "pending | in_progress | ready | not_applicable", "updated_at": "ISO8601 | null" },
    "prototype":     { "status": "pending | in_progress | ready", "updated_at": "ISO8601 | null", "error": "string | null" }
  },
  "screens": [
    { "screen_id": "string", "screen_name": "string", "route": "string" }
  ],
  "byproducts": {
    "requirements": {
      "product_name": "string | null",
      "screen_count": 0,
      "persona_count": 0,
      "flow_count": 0,
      "screens": [{ "screen_id": "string", "screen_name": "string", "priority": "p0 | p1 | p2", "description": "string" }],
      "personas": [{ "id": "string", "name": "string" }],
      "flows": [{ "flow_id": "string", "flow_name": "string" }],
      "constraints": [{ "type": "string", "value": "string" }],
      "analyst_notes": []
    },
    "flow":    { "present": false },
    "journey": { "present": false },
    "lo_fi":   { "present": false, "screen_count": 0 }
  },
  "artifacts": [
    { "name": "string", "file": "string", "description": "string", "present": true }
  ]
}
```

**Manifest update pattern** (used by all agents):
```
1. Read existing manifest
2. Merge only your fields — do not touch other agents' fields
3. Write the whole file back
```

If `spa/public/pipeline-manifest.json` does not exist (shell-scaffolder hasn't run yet), skip the update.

---

## `requirements.json`

Written by `requirements-analyst`.

```json
{
  "meta": {
    "project_name": "string",
    "version": "1.0.0",
    "created_at": "ISO8601",
    "source_mode": "brief | figma | seed | combined",
    "framework": "react | vue"
  },
  "product": {
    "name": "string",
    "description": "string",
    "platform_targets": ["web"]
  },
  "personas": [
    {
      "id": "string",
      "name": "string",
      "goal": "string",
      "primary_flow": "flow_id"
    }
  ],
  "screens": [
    {
      "screen_id": "string",
      "screen_name": "string",
      "description": "string",
      "states": ["default", "loading", "error"],
      "entry_points": ["screen_id"],
      "exit_points": ["screen_id"],
      "mock_data_needed": ["string"],
      "priority": "p0 | p1 | p2",
      "notes": ""
    }
  ],
  "flows": [
    {
      "flow_id": "string",
      "flow_name": "string",
      "flow_type": "primary | secondary | error",
      "screens": ["screen_id"]
    }
  ],
  "design_tokens": {
    "tokens_source": "provided | hints | none",
    "token_file_path": "string | null",
    "hints": { "colors": {}, "typography": {}, "spacing": {} }
  },
  "constraints": [
    { "type": "platform | accessibility | framework | hard_rule", "value": "string", "source": "brief | seed | inferred" }
  ],
  "analyst_notes": []
}
```

---

## `screen-map.json`

Written by `screen-mapper`.

```json
{
  "meta": {
    "generated_from": "requirements.json",
    "generated_at": "ISO8601",
    "total_screens": 0,
    "framework": "react | vue"
  },
  "screens": [
    {
      "screen_id": "string",
      "screen_name": "string",
      "screen_index": 0,
      "route": "/path",
      "description": "string",
      "priority": "p0 | p1 | p2",
      "is_entry_point": false,
      "is_modal": false,
      "states": ["default", "loading", "error"],
      "layout": {
        "header": {
          "present": true,
          "slots": [
            { "slot_id": "string", "slot_type": "string", "label": "string", "optional": false, "component_hint": "string" }
          ]
        },
        "content": { "scrollable": true, "slots": [] },
        "footer": { "present": false, "slots": [] },
        "overlay": null
      },
      "navigation": [
        {
          "to": "screen_id",
          "trigger": "tap_button | tap_card | submit_form | swipe_back | auto",
          "trigger_element": "slot_id | null",
          "condition": "string | null",
          "is_happy_path": true
        }
      ],
      "mock_data": [
        {
          "entity_id": "string",
          "fields": [{ "name": "string", "type": "string | number | boolean | array", "example_value": "string" }],
          "count": 3
        }
      ]
    }
  ],
  "entry_screen": "screen_id",
  "flows": [
    { "flow_id": "string", "flow_name": "string", "flow_type": "primary | secondary | error", "screen_sequence": ["screen_id"] }
  ],
  "mapper_notes": []
}
```

---

## `flow-index.json`

Written by `flow-diagram-builder`.

```json
{
  "meta": {
    "generated_from": "screen-map.json",
    "generated_at": "ISO8601",
    "total_screens": 0,
    "excalidraw_file": "flow.excalidraw"
  },
  "screens": [
    { "screen_id": "string", "screen_name": "string", "node_element_id": "string", "canvas_position": { "x": 0, "y": 0 } }
  ]
}
```

---

## `journey-index.json`

Written by `journey-diagram-builder`.

```json
{
  "meta": {
    "generated_from": "requirements.json + screen-map.json",
    "generated_at": "ISO8601",
    "total_personas": 0,
    "total_steps": 0,
    "excalidraw_file": "journey.excalidraw"
  },
  "personas": [
    {
      "persona_id": "string",
      "persona_name": "string",
      "steps": [{ "step_index": 0, "label": "string", "phase": "string", "score": 4 }]
    }
  ],
  "insights": ["string"]
}
```

---

## `lo-fi-index.json`

Written by `lo-fi-wireframe-builder`.

```json
{
  "meta": {
    "generated_from": "screen-map.json",
    "generated_at": "ISO8601",
    "total_screens": 0,
    "excalidraw_file": "lo-fi.excalidraw",
    "svg_fallback": null
  },
  "screens": [
    {
      "screen_id": "string",
      "screen_name": "string",
      "frame_element_id": "string",
      "slot_element_ids": { "slot_id": "element_id" },
      "canvas_position": { "x": 0, "y": 0 }
    }
  ]
}
```

---

## `spa-manifest.json`

Written by `spa-generator`.

```json
{
  "meta": {
    "generated_at": "ISO8601",
    "framework": "react | vue",
    "output_dir": "/absolute/path",
    "total_views": 0,
    "total_components": 0,
    "design_system_applied": {
      "mode": "none | css | json_tokens | figma | tailwind | named | description",
      "source": "string | null",
      "tokens_css_written": false,
      "fallback_used": false,
      "fallback_reason": null
    },
    "lo_fi_gated": false
  },
  "files": [
    { "path": "spa/src/views/HomeView.jsx", "type": "view", "screen_id": "home" },
    { "path": "spa/src/components/NavBar.jsx", "type": "component" },
    { "path": "spa/src/data/mock-data.json", "type": "data" }
  ],
  "routes": [
    { "path": "/", "view": "HomeView", "screen_id": "home" }
  ]
}
```

---

## `validation-report.json`

Written by `prototype-validator`.

```json
{
  "meta": {
    "generated_at": "ISO8601",
    "framework": "react | vue",
    "parity_score": 0.95,
    "result": "PASS | FAIL",
    "gate_threshold": 0.90
  },
  "checks": [
    {
      "check_id": "route_coverage",
      "name": "Route Coverage",
      "weight": 0.25,
      "result": "PASS | FAIL | WARN",
      "score": 1.0,
      "findings": []
    }
  ],
  "blocking_issues": [
    {
      "check_id": "string",
      "severity": "error | warning",
      "message": "string",
      "file": "string | null",
      "fix": "string"
    }
  ],
  "validator_notes": []
}
```

---

## `delivery-package.json`

Written by `delivery-sequencer`.

```json
{
  "meta": {
    "project_name": "string",
    "run_id": "string",
    "completed_at": "ISO8601",
    "framework": "react | vue",
    "parity_score": 0.95,
    "lo_fi_produced": false
  },
  "artifacts": [
    {
      "artifact_id": "string",
      "path": "relative to working_dir",
      "type": "config | requirements | screen_map | flow_diagram | user_journey | lo_fi | spa | validation | delivery",
      "description": "string",
      "required": true,
      "present": true
    }
  ],
  "screens": [
    { "screen_id": "string", "screen_name": "string", "route": "string", "view_file": "spa/src/views/..." }
  ],
  "run_instructions": {
    "install": "cd <working_dir>/spa && npm install",
    "dev": "npm run dev",
    "url": "http://localhost:5173",
    "static_open": "open <working_dir>/spa/index.html (Vue hash history only)"
  },
  "delivery_notes": []
}
```
