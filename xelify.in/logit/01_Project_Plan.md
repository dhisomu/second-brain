# Project Plan: Logit - Universal Paperless Log System

This document outlines the development plan for Logit using a structured approach: **EPICs → Features → User Stories**.

---

## 🏗 EPIC 1: Core Engine & Data Persistence (Backend & DB)
**Goal**: Establish a robust, high-performance foundation to store dynamic form definitions and industrial log data.

### Feature 1.1: Dynamic Form Schema Management
*   **User Story**: As an Admin, I want to save and version my form designs so that I can evolve my logging process without losing old data.
*   **Acceptance Criteria**:
    *   **Given**: A valid JSON form schema.
    *   **When**: I call the `/api/forms/publish` endpoint.
    *   **Then**: The system saves the schema to the `FormDefinitions` table and increments the version number.
*   **Test Case (AAA)**:
    *   **Arrange**: Create a mock JSON schema for a "Daily Check".
    *   **Act**: Post the schema to the API.
    *   **Assert**: Verify status code `201` and check if `version` is `1`.

### Feature 1.2: JSONB Data Submission
*   **User Story**: As a Field Operator, I want to submit my log entry so that it is stored securely for future audits.
*   **Acceptance Criteria**:
    *   **Given**: A completed dynamic form.
    *   **When**: I click "Submit".
    *   **Then**: The backend validates the data against the current schema and stores it in the `LogSubmissions` table.
*   **Test Case (AAA)**:
    *   **Arrange**: Prepare a submission payload `{"temp": 80}` matching a form schema.
    *   **Act**: Send a POST request to `/api/logs/submit`.
    *   **Assert**: Confirm the record exists in the database with the correct `form_definition_id`.

---

## 🎨 EPIC 2: Form Builder & Visual Canvas (Admin UI)
**Goal**: Provide a "Power Apps" style drag-and-drop experience for creating complex forms.

### Feature 2.1: Visual Grid Canvas (Snap-to-Grid)
*   **User Story**: As a Process Manager, I want to drag components onto a grid so that I can design a clean, professional layout for my operators.
*   **Acceptance Criteria**:
    *   **Given**: The Form Builder screen with a 12-column grid.
    *   **When**: I drag a "Text" component near a grid line.
    *   **Then**: The component snaps to the nearest grid coordinates and saves its `row/column` metadata.
*   **Test Case (AAA)**:
    *   **Arrange**: Initialize the `interact.js` canvas.
    *   **Act**: Simulate a drag of a component to `x: 15px, y: 15px`.
    *   **Assert**: Verify the component coordinates are adjusted to the grid snap point (e.g., `0px, 0px`).

### Feature 2.2: Property Inspector
*   **User Story**: As an Admin, I want to configure specific validation rules for a field so that I can ensure operators enter correct data.
*   **Acceptance Criteria**:
    *   **Given**: A selected "Number" component on the canvas.
    *   **When**: I enter "0" in Min and "100" in Max in the sidebar.
    *   **Then**: The underlying JSON schema for that component is updated with the validation rules.

---

## 🚀 EPIC 3: Dynamic Form Rendering & Logic (Field UI)
**Goal**: Render high-performance, responsive forms on any device with complex real-time logic.

### Feature 3.1: Cascading Filter Engine (Place → Plant)
*   **User Story**: As an Operator, I want the "Plant" list to filter automatically based on my "Place" selection so that I don't pick the wrong equipment.
*   **Acceptance Criteria**:
    *   **Given**: A form with a Place dropdown and a Plant dropdown.
    *   **When**: I select "Vattamalai" as the Place.
    *   **Then**: The Plant dropdown only displays plants associated with Vattamalai.
*   **Test Case (AAA)**:
    *   **Arrange**: Select "Vattamalai" in the UI.
    *   **Act**: Trigger the `onChange` event for the Place field.
    *   **Assert**: Check that the Plant dropdown options count matches the master data for that location.

---

## 🛡️ EPIC 4: Workflow & Audit (Management)
**Goal**: Ensure full traceability and approval cycles for every log entry.

### Feature 4.1: Immutable Audit Trail
*   **User Story**: As an Auditor, I want to see the history of changes for a log entry so that I can verify data integrity.
*   **Acceptance Criteria**:
    *   **Given**: An existing log entry.
    *   **When**: A manager edits the status from "Submitted" to "Approved".
    *   **Then**: An entry is created in the audit log showing the old value, new value, timestamp, and user.

---

## 🖥️ Screen Components Inventory

### 1. Form Builder (Admin)
- `CanvasGrid`: The main drop zone with 12/24 column support.
- `ComponentPallete`: Sidebar containing all available Field Types (Input, Media, etc.).
- `PropertyInspector`: Contextual sidebar to edit selected component settings.
- `TemplateSelector`: Modal to pick from industry-standard templates (8D, PDCA).

### 2. Form Renderer (Operator)
- `DynamicFormContainer`: Loads and renders the entire JSON schema.
- `FieldRenderer`: A generic component that switches UI based on `subType` (e.g., Phone, Date).
- `ValidationToast`: Real-time feedback overlay for errors.
- `MediaUploader`: Handles images and PDF uploads with progress bars.

---

## 💾 Backend & Database Planning

### Database Schema (PostgreSQL)
- **`form_definitions`**: Relational metadata + `schema` (JSONB).
- **`log_submissions`**: Relational metadata + `data` (JSONB).
- **`master_data`**: Specialized tables for high-performance lookup (e.g., `plants`, `places`).
- **`audit_logs`**: Immutable table for change tracking.

### API Structure (FastAPI)
- `POST /api/forms`: Create a new form version.
- `GET /api/forms/{id}`: Fetch form schema for rendering.
- `POST /api/logs`: Submit log data.
- `GET /api/logs/{id}/audit`: Fetch change history.
- `GET /api/lookup/{category}`: Fetch master data for cascading dropdowns.

---
*Last Updated: May 6, 2026*
