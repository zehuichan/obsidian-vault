---
url: "https://skills.sh/github/awesome-copilot/excalidraw-diagram-generator"
clipped: 2026-08-11
title: "excalidraw-diagram-generator — Agent Skill"
---

# excalidraw-diagram-generator

Generate Excalidraw diagrams from natural language descriptions in multiple formats.

**Install:**

```bash
npx skills add https://github.com/github/awesome-copilot --skill excalidraw-diagram-generator
# or
npx skills add github/awesome-copilot@excalidraw-diagram-generator -g -y
```

**Stats (as of clip):** ~27.6K installs on skills.sh; source repo github/awesome-copilot (~37.7K stars). First seen Feb 4, 2026.

## Summary (skills.sh)

- Supports nine diagram types: flowcharts, relationship diagrams, mind maps, architecture diagrams, data flow diagrams, swimlane business flows, class diagrams, sequence diagrams, and ER diagrams
- Outputs valid `.excalidraw` JSON files that open directly in Excalidraw or the VS Code extension
- Includes layout guidelines, element count recommendations, and color schemes for consistent visual design
- Optional integration with icon libraries (AWS, GCP, Azure, etc.) via Python scripts for professional architecture diagrams
- Provides structured workflows for extracting diagram requirements, choosing appropriate types, and generating properly formatted output

## SKILL.md (installed copy excerpt)

```yaml
name: excalidraw-diagram-generator
description: 'Generate Excalidraw diagrams from natural language descriptions. Use when asked to "create a diagram", "make a flowchart", "visualize a process", "draw a system architecture", "create a mind map", or "generate an Excalidraw file". Supports flowcharts, relationship diagrams, mind maps, and system architecture diagrams. Outputs .excalidraw JSON files that can be opened directly in Excalidraw.'
```

### When to Use

- "Create a diagram showing..."
- "Make a flowchart for..."
- "Visualize the process of..."
- "Draw the system architecture of..."
- "Generate a mind map about..."
- "Create an Excalidraw file for..."
- "Show the relationship between..."
- "Diagram the workflow of..."

### Supported diagram types

- Flowcharts — sequential processes, workflows, decision trees
- Relationship Diagrams — entity relationships, system components, dependencies
- Mind Maps — concept hierarchies, brainstorming, topic organization
- Architecture Diagrams — system design, module interactions, data flow
- Data Flow Diagrams (DFD) — data flow visualization, transformations
- Business Flow (Swimlane) — cross-functional workflows, actor-based process flows
- Class Diagrams — OOP class structures and relationships
- Sequence Diagrams — object interactions over time, message flows
- ER Diagrams — database entity relationships, data models

### Workflow

1. Understand the request (type, key elements, relationships, complexity)
2. Choose diagram type from intent/keywords table
3. Extract structured information for that type
4. Generate Excalidraw JSON (`rectangle`, `ellipse`, `diamond`, `arrow`, `text`; all text uses `fontFamily: 5` / Excalifont)
5. Wrap in file format:

```json
{
  "type": "excalidraw",
  "version": 2,
  "source": "https://excalidraw.com",
  "elements": [],
  "appState": {
    "viewBackgroundColor": "#ffffff",
    "gridSize": 20
  },
  "files": {}
}
```

6. Save as `<descriptive-name>.excalidraw` and instruct opening via https://excalidraw.com (Open / drag-drop) or VS Code Excalidraw extension

### Package layout (installed)

```
excalidraw-diagram-generator/
├── SKILL.md
├── references/
│   ├── element-types.md
│   └── excalidraw-schema.md
├── scripts/
│   ├── add-arrow.py
│   ├── add-icon-to-diagram.py
│   ├── split-excalidraw-library.py
│   └── README.md
└── templates/
    ├── flowchart-template.excalidraw
    ├── relationship-template.excalidraw
    ├── mindmap-template.excalidraw
    ├── data-flow-diagram-template.excalidraw
    ├── business-flow-swimlane-template.excalidraw
    ├── class-diagram-template.excalidraw
    ├── sequence-diagram-template.excalidraw
    └── er-diagram-template.excalidraw
```

### Layout / color guidelines (from skill)

- Horizontal gap 200–300px; vertical gap 100–150px
- Primary: `#a5d8ff`; Secondary: `#b2f2bb`; Central: `#ffd43b`; Alerts: `#ffc9c9`
- Text 16–24px; always `fontFamily: 5`
- Recommended element counts (e.g. flowchart 3–10 steps, mind map 4–6 branches)

### Opening generated files

- Visit https://excalidraw.com — Open or drag-and-drop the `.excalidraw` file
- Or use the Excalidraw VS Code extension

## Related product

Hosted whiteboard: https://excalidraw.com/
GitHub: https://github.com/excalidraw/excalidraw
