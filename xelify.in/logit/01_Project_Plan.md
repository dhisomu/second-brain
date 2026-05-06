# Project Plan: Logit - Universal Paperless Log System

This document outlines the development plan for Logit using a structured approach: **EPICs → Features → User Stories**.

---

## 👥 User Personas & Journeys

To ensure strict data isolation and organizational control, the platform recognizes four distinct personas.

### 1. Super Admin (IT Team)
- **Role**: System Governance.
- **Journey**: Creates Departments, assigns **Department Key Users**, and initializes "Form Groups" for each department.
- **Access**: Global visibility for technical support and compliance.

### 2. Department Key User (Dept Admin)
- **Role**: The "Process Owner" for a specific department (e.g., Quality, Production).
- **Journey**: Designs forms, defines approval workflows, and manages User Groups *within their department*.
- **Constraint**: **Isolation**. A Quality Key User cannot see or edit Production form templates.

### 3. Field Operator (Creator)
- **Role**: Data entry on the shop floor.
- **Journey**: Scans a QR code or selects a form from their dashboard, fills in data, and submits for approval.

### 4. Approver (Manager/Lead)
- **Role**: Quality gatekeeper.
- **Journey**: Receives notifications for pending logs, reviews the data, and either Approves or Rejects with comments.

---

---

## 🗓️ Development Execution Order (The Build Sequence)

To ensure a functional system as quickly as possible, development will follow this prioritized sequence:

### <a id="phase-1"></a>Phase 1: The Foundation (Backend & Database) - **Priority: HIGH**
*Build the "Brains" first so data can be saved and retrieved.*
1.  **DB Schema Setup**: Create `FormDefinitions` and `LogSubmissions` tables with JSONB support. ([Refer Tech Details](#tech-details))
2.  **Schema APIs**: Develop endpoints to save (POST) and fetch (GET) form metadata.
3.  **Submission APIs**: Develop endpoints to receive and validate log data from operators.
4.  **RBAC & User Groups**: Implement the database and UI for managing user roles, groups, and approval permissions. ([Refer Epic 5](#epic-5))

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

### Feature 4.1: Comprehensive Lifecycle Audit Trail
*   **User Story**: As an Auditor, I want to see the full history of a log (Submission, Approvals, Edits, Resubmissions) so that I can verify the complete chain of custody.
*   **Acceptance Criteria**:
    *   **Given**: A log entry.
    *   **When**: Any action is taken (Submit, Approve, Reject, Edit, Resubmit).
    *   **Then**: A new entry is added to `AuditLogs` with the action name and a JSONB delta of what changed.

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

## <a id="epic-5"></a>👥 EPIC 5: RBAC & User Group Management (Phase 1)
[← Back to Phase 1](#phase-1)
**Goal**: Establish a secure permission system to control who can create, edit, and approve logs.

### Feature 5.1: Multi-Stage Approval Configuration
*   **User Story**: As an Admin, I want to define if a form requires 1 or 2 stages of approval so that I can enforce quality standards for critical logs.
*   **Acceptance Criteria**:
    *   **Given**: The Form Builder metadata section.
    *   **When**: I set `approvals_required` to `2` and assign `UserGroup: QualityManager`.
    *   **Then**: The form schema saves these workflow rules in the `workflow_config` JSONB column.
*   **Test Case (AAA)**:
    *   **Arrange**: Define a form with a 2-stage approval workflow.
    *   **Act**: Save the form definition.
    *   **Assert**: Verify `workflow_config->'approvals_required'` equals `2`.

### Feature 5.2: User Group & Role Assignment UI
*   **User Story**: As an Admin, I want to group users and assign roles (Creator, Editor, Approver) so that I can manage permissions at scale.
*   **Acceptance Criteria**:
    *   **Given**: The Admin Dashboard.
    *   **When**: I create a group "Production Team" and add 5 users with the "Operator" role.
    *   **Then**: The system stores this mapping in the `UserGroups` table.

---

## 💾 Technical Deep Dive: Database Schema Details
[← Back to Phase 1](#phase-1)

To implement Phase 1, the following PostgreSQL structures will be established using **SQLModel** (FastAPI-ready ORM).

### 1. `FormDefinitions` (The Blueprint)
This table stores the master configuration of every form created in the system.

| Column | Type | Description |
|:---|:---|:---|
| `form_id` | `UUID` | **Auto-generated** unique identifier (with built-in deduplication). |
| `name` | `String` | **Unique** human-readable name (e.g., "Daily Shift Log"). |
| `department_id` | `String` | **Isolation Key**: Links form to a specific department (e.g., "DEPT-QA"). |
| `category_id` | `String` | **Internal Grouping**: e.g., "Maintenance", "Safety" within a department. |
| `slug` | `String` | **Unique** URL-friendly ID (e.g., "daily-shift-log"). |
| `version` | `Integer` | Auto-increments when the schema is updated. |
| **`schema`** | **`JSONB`** | **The Core**: Stores fields, types, validation, and grid layout. |
| `workflow_config` | `JSONB` | Stores approval rules (e.g., `{"stages": 2, "approvers": ["group_id"]}`). |
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
| `status` | `String` | Workflow status (Draft, Submitted, Approved, Rejected). |
| `approved_by_1` | `String` | User ID of the first-stage approver. |
| `approved_by_2` | `String` | User ID of the second-stage approver. |
| `approval_1_at` | `DateTime` | Timestamp of first approval. |
| `approval_2_at` | `DateTime` | Timestamp of second approval. |
| `submitted_at`| `DateTime` | Timestamp of submission. |
| `submitted_by`| `String` | User who entered the data. |
| `metadata` | `JSONB` | Stores GPS, IP address, and browser/OS info. |

### 3. `AuditLogs` (The History)
A read-only table for immutable tracking of all lifecycle events.

| Column | Type | Description |
|:---|:---|:---|
| `id` | `BigInt` | Sequence ID. |
| `target_id` | `UUID` | Reference to the `LogSubmissions` entry being changed. |
| `old_value` | `JSONB` | Snapshot of data/status **before** the action. |
| `new_value` | `JSONB` | Snapshot of data/status **after** the action. |
| `action` | `String` | Lifecycle event: `SUBMIT`, `APPROVE_1`, `APPROVE_2`, `REJECT`, `EDIT_RESUBMIT`. |
| `performed_by`| `String` | User who took the action. |
| `comments` | `String` | Optional notes (e.g., "Reason for rejection"). |
| `timestamp` | `DateTime` | When the event occurred. |

### 4. Implementation Example (Concrete Data Case Study)

This example tracks the journey of two different forms through their lifecycle.

#### **Scenario A: 1-Stage Approval (Shift Handover)**
*Created: 08:00 AM | Submitted: 04:00 PM | Approved: 04:30 PM*

**Table: `FormDefinitions`**
| form_id | name | version | workflow_config | created_at |
|:---|:---|:---|:---|:---|
| `f-100` | Shift Handover | 1 | `{"stages": 1, "can_create": ["g-ops"], "can_edit": ["g-ops"], "approvers_s1": ["g-ops-mgr"]}` | `2026-05-06T08:00:00Z` |

**Table: `LogSubmissions`**
| log_id | form_id | version | data (JSONB) | status | submitted_at | approved_by_1 |
|:---|:---|:---|:---|:---|:---|:---|
| `L-001` | `f-100` | 1 | `{"shift": "Morning"}` | `Approved` | `2026-05-06T16:00:00Z` | `mgr_smith@xelify.in` |

---

#### **Scenario B: 2-Stage Approval (Quality Audit)**
*Created: 09:00 AM | Submitted: 10:00 AM | Stage 1 Approved: 11:00 AM | Stage 2 Approved: 02:00 PM*

**Table: `FormDefinitions`**
| form_id | name | version | workflow_config | created_at |
|:---|:---|:---|:---|:---|
| `f-200` | Quality Audit | 1 | `{"stages": 2, "can_create": ["g-ops"], "can_edit": ["g-ops"], "approvers_s1": ["g-qa"], "approvers_s2": ["g-factory-mgr"]}` | `2026-05-06T09:00:00Z` |

**Table: `LogSubmissions`**
| log_id | form_id | status | approved_by_1 | approved_by_2 | approval_1_at | approval_2_at |
|:---|:---|:---|:---|:---|:---|:---|
| `L-002` | `f-200` | `Approved` | `lead_qa@xelify.in` | `factory_mgr@xelify.in` | `11:00:00Z` | `14:00:00Z` |

---

#### **Table: `AuditLogs` (Rejection & Resubmission Scenario)**
*Example: A manager rejects a log for missing data, and the operator resubmits it.*

| id | target_id | action | old_value | new_value | comments | performed_by | timestamp |
|:---|:---|:---|:---|:---|:---|:---|:---|
| 10 | `L-003` | `SUBMIT` | `{}` | `{"status": "Submitted"}` | "Initial log" | `op_alice@xelify.in` | `10:00:00Z` |
| 11 | `L-003` | `REJECT` | `{"status": "Submitted"}` | `{"status": "Rejected"}` | "Missing data" | `mgr_bob@xelify.in` | `10:30:00Z` |
| 12 | `L-003` | `EDIT_RESUBMIT` | `{"temp": null}` | `{"temp": 36.5, "status": "Submitted"}` | "Added data" | `op_alice@xelify.in` | `11:00:00Z` |
| 13 | `L-003` | `APPROVE_1` | `{"status": "Submitted"}` | `{"status": "Approved"}` | "Verified" | `mgr_bob@xelify.in` | `11:15:00Z` |

#### **Table: `UserGroups`**
| group_id | group_name | members (JSONB) | permissions (JSONB) |
|:---|:---|:---|:---|
| `g-qa` | QA Leads | `["lead_qa@xelify.in"]` | `{"can_approve": true, "stage": 1}` |
| `g-mgr` | Factory Mgmt | `["factory_mgr@xelify.in"]` | `{"can_approve": true, "stage": 2}` |

---
*Last Updated: May 6, 2026*
