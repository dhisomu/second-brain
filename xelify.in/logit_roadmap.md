# Logit Roadmap: Universal Paperless Log System

## 1. Project Vision
Logit aims to be a high-performance, low-code platform for creating digital logbooks. It transforms physical data collection into a structured, searchable, and secure digital asset.

## 2. Strategic Roadmap

### Phase 1: Foundation (Completed/In Progress)
- [x] Triple-Stack Infrastructure Setup (Dev/Stage/Prod).
- [x] Basic FastAPI Backend with PostgreSQL.
- [x] Simple Form Configurator (Property-list based).
- [x] Dynamic Form Rendering and Data Entry.
- [x] Passwordless OTP Authentication.

### Phase 2: Enhanced Component Architecture (Current)
- **Advanced Schema Definition**: Transition from simple label/type objects to rich component definitions.
- **Details Screen Components**: Implementation of specialized UI components with advanced validation, events, and data binding.
- **Search & Filter Integration**: Adding functional components like the "Search Bar" to forms/dashboards.

### Phase 3: Visual Form Builder (The "Canvas")
- **Grid Layout System**: A "Draw.io" style canvas for form design.
- **Snap-to-Grid Positioning**: Drag-and-drop fields with visual alignment guides.
- **Responsive Layout Control**: Ability to define column widths (e.g., 1/2, 1/3, full width) visually.
- **Interactive Component Tree**: Hierarchical view of form components.

### Phase 4: Workflow & Logic
- **Conditional Logic**: Show/Hide fields based on other field values.
- **Calculated Fields**: Real-time math (e.g., Summing parts cost, calculating averages).
- **Multi-step Forms**: Support for wizards and multi-page logs.
- **Action Triggers**: Webhooks and email notifications upon submission.

### Phase 5: Analytics & Data Intelligence
- **Submission Dashboard**: Visual charts for logged data.
- **Advanced Export**: Scheduled PDF/Excel reports sent to stakeholders.
- **Offline Support**: PWA capabilities for logging data without internet.

---

## 3. Core Component Specification (Details Screen)

The next generation of Logit components will follow a unified JSON schema to support advanced features like debounce, regex validation, and event handling.

### Example: Search Bar Component
```json
{
  "id": "search_bar",
  "name": "Search Bar",
  "type": "input",
  "subType": "text",
  "label": "Search",
  "placeholder": "Search assets...",
  "mandatory": false,
  "visible": true,
  "editable": true,
  "defaultValue": "",
  "helpText": "Type asset name or keyword",
  "validation": {
    "minLength": 0,
    "maxLength": 100,
    "regex": "^[a-zA-Z0-9 _-]*$",
    "allowSpecialCharacters": false,
    "trimInput": true,
    "caseSensitive": false
  },
  "errorMessages": {
    "maxLength": "Maximum 100 characters allowed",
    "regex": "Only letters, numbers, spaces, _ and - allowed"
  },
  "events": [
    {
      "event": "onChange",
      "action": "filterList",
      "debounce": 300
    }
  ],
  "dataBinding": {
    "field": "assetName",
    "operation": "contains"
  },
  "permissions": ["Admin", "User", "Viewer"],
  "uiPosition": {
    "section": "header",
    "order": 1
  }
}
```

---

## 4. Visual Builder Experience (Low-Code Canvas)

To provide a "Power Apps" feel, the builder will move from a vertical list to a grid-based coordinate system.

| Feature | Description |
| :--- | :--- |
| **Grid Overlay** | 12-column or 24-column CSS Grid background for alignment. |
| **Snap-to-Grid** | Components automatically snap to the nearest grid line when moved or resized. |
| **Absolute/Relative Toggle** | Flexibility to use standard flow layout or absolute positioning for pixel-perfect forms. |
| **Property Inspector** | Sidebar to edit the advanced JSON properties (validation, events) of the selected field. |
| **Real-time Preview** | Toggle between "Design" and "Live" mode instantly. |

---

## 5. Technical Implementation Path

1. **Schema Migration**: Update backend models to store the rich JSON metadata in a `JSONB` column.
2. **Component Registry**: Create a Javascript library of renderers for each `subType`.
3. **Canvas Engine**: Implement `interact.js` or custom drag-logic for the grid-based builder.
4. **Validation Engine**: Centralize regex and constraint checking on both frontend (UI) and backend (API).
