# Project Plan: Logit - Universal Paperless Log System

This document outlines the development plan for Logit using a structured approach: **EPICs → Features → User Stories**.

---

## 🗓️ Development Execution Order (The Build Sequence)

To ensure a functional system as quickly as possible, development will follow this prioritized sequence:

### <a id="phase-1"></a>Phase 1: The Foundation (Backend & Database) - **Priority: HIGH**
*Build the "Brains" first so data can be saved and retrieved.*
1.  **DB Schema Setup**: Create `FormDefinitions` and `LogSubmissions` tables with JSONB support. ([Refer Tech Details](#tech-details))
2.  **Schema APIs**: Develop endpoints to save (POST) and fetch (GET) form metadata.
3.  **Submission APIs**: Develop endpoints to receive and validate log data from operators.

### <a id="phase-2"></a>Phase 2: The Dynamic Renderer (Field UI) - **Priority: HIGH**
*Make it possible to display forms from the database.*
1.  **Field Library**: Create the JS logic to render each input type (Text, Number, Phone, etc.).
2.  **Dynamic Container**: Build the "Engine" that reads a JSON schema and renders the full form UI.
3.  **Client-side Validation**: Implement the regex and min/max logic in the browser. ([Refer Epic 3](#epic-3))

### <a id="phase-3"></a>Phase 3: The Visual Builder (Admin Canvas) - **Priority: MEDIUM**
*Build the user-friendly interface to create the forms.*
1.  **Grid Canvas**: Implement the snap-to-grid visual area.
2.  **Component Palette**: Enable dragging elements onto the canvas.
3.  **Property Inspector**: The sidebar for configuring field rules (Validation, Place/Plant filters). ([Refer Epic 2](#epic-2))

### <a id="phase-4"></a>Phase 4: Workflow & Governance (Management) - **Priority: LOW**
*Finalize the industrial-grade features.*
1.  **Role-Based Access**: Implement Admin/Operator/Manager permissions.
2.  **Audit Trail Engine**: Immutable logging of all changes.
3.  **Reporting Dashboard**: Tabular view and Excel/PDF export of submitted logs. ([Refer Epic 4](#epic-4))

---

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

## <a id="epic-2"></a>🎨 EPIC 2: Form Builder & Visual Canvas (Admin UI)
[← Back to Phase 3](#phase-3)
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

## <a id="epic-3"></a>🚀 EPIC 3: Dynamic Form Rendering & Logic (Field UI)
[← Back to Phase 2](#phase-2)
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

## <a id="epic-4"></a>🛡️ EPIC 4: Workflow & Audit (Management)
[← Back to Phase 4](#phase-4)
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

## <a id="tech-details"></a>💾 Technical Deep Dive: Database Schema Details
[← Back to Phase 1](#phase-1)

To implement Phase 1, the following PostgreSQL structures will be established using **SQLModel** (FastAPI-ready ORM).

### 1. `FormDefinitions` (The Blueprint)
This table stores the master configuration of every form created in the system.

| Column | Type | Description |
|:---|:---|:---|
| `form_id` | `UUID` | **Auto-generated** unique identifier (with built-in deduplication). |
| `name` | `String` | **Unique** human-readable name (e.g., "Daily Shift Log"). |
| `slug` | `String` | **Unique** URL-friendly ID (e.g., "daily-shift-log"). |
| `version` | `Integer` | Auto-increments when the schema is updated. |
| **`schema`** | **`JSONB`** | **The Core**: Stores fields, types, validation, and grid layout. |
| `is_active` | `Boolean` | Controls whether operators can see the form. |
| `created_at` | `DateTime` | Timestamp of creation. |
| `created_by` | `String` | User email/ID who created the form. |

### 2. `LogSubmissions` (The Data)
This table stores the entries pushed by operators in the field.

| Column | Type | Description |
|:---|:---|:---|
| `log_id` | `UUID` | **Auto-generated** unique identifier for the log entry. |
| `form_id` | `UUID` | Reference to the `FormDefinitions` entry. |
| `version` | `Integer` | **Snapshot** of the `version` from `FormDefinitions` used at submission time. |
| **`data`** | **`JSONB`** | **The Payload**: Key-value pairs of operator input. |
| `status` | `String` | Workflow status (Draft, Submitted, Approved). |
| `submitted_at`| `DateTime` | Timestamp of submission. |
| `submitted_by`| `String` | User who entered the data. |
| `metadata` | `JSONB` | Stores GPS, IP address, and browser/OS info. |

### 4. Implementation Example (Concrete Data)

To visualize how these tables work together, here is a "Simple Feedback" form example.

#### **Table: `FormDefinitions` (The Blueprint)**
| form_id | name | version | schema (JSONB) |
|:---|:---|:---|:---|
| `uuid-1` | Simple Feedback | 1 | `{"fields": [{"type": "ui", "subType": "label", "text": "Feedback Form"}, {"id": "user_name", "type": "input", "subType": "text", "label": "Your Name"}, {"id": "submit_btn", "type": "action", "subType": "button", "label": "Submit"}]}` |

#### **Table: `LogSubmissions` (The Data)**
Two different operators submit data using the blueprint above.

| log_id | form_id | version | data (JSONB) | submitted_by |
|:---|:---|:---|:---|:---|
| `log-101` | `uuid-1` | 1 | `{"user_name": "Alice Walker"}` | operator_alice@xelify.in |
| `log-102` | `uuid-1` | 1 | `{"user_name": "Bob Smith"}` | operator_bob@xelify.in |

#### **Table: `AuditLogs` (The History)**
If a manager corrects Bob's name later.

| id | target_id (log_id) | old_value (JSONB) | new_value (JSONB) | action |
|:---|:---|:---|:---|:---|
| 1 | `log-102` | `{"user_name": "Bob Smith"}` | `{"user_name": "Robert Smith"}` | VALUE_EDIT |

---
*Last Updated: May 6, 2026*
