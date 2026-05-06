# Logit Roadmap: Universal Paperless Log System

## 1. Project Vision
Logit is a dynamic form-based logging platform that enables users to create customizable forms for operational logging, inspections, audits, compliance tracking, maintenance activities, industrial data collection, and workflow approvals.

Logit transforms physical data collection into a structured, searchable, and secure digital asset, allowing non-technical users to design forms visually and capture data efficiently through web and mobile interfaces.

---

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
- **Responsive Layout Control**: Ability to define column widths and responsive grid positions visually.
- **Interactive Component Tree**: Hierarchical view of form components.

### Phase 4: Workflow & Logic
- **Conditional Logic**: Show/Hide fields based on other field values (e.g., `showIf` machine failure = true).
- **Calculated Fields**: Real-time math (e.g., Summing parts cost, calculating averages).
- **Multi-step Forms**: Support for wizards and multi-page logs.
- **Workflow Engine**: Status management (Draft, Submitted, Under Review, Approved).

### Phase 5: Analytics & Data Intelligence
- **Submission Dashboard**: Visual charts for logged data.
- **Advanced Export**: Scheduled PDF/Excel reports sent to stakeholders.
- **Offline Support**: PWA capabilities for logging data without internet.
- **IoT Integration**: Support for sensors, Digital Twins, and industrial protocols (BACnet/Modbus).

---

## 3. Key Modules

### 3.1 Form Builder Module
- **Create/Edit/Clone**: Full lifecycle management of forms.
- **Version Control**: Tracking changes and versioning for compliance.
- **Categorization**: Organize forms by category or industrial module.
- **Publishing**: Managed lifecycle for deploying forms to field operators.
- **Templates Library**: Pre-built industry-standard templates to accelerate form creation:
    - **Quality & Root Cause**: Why-Why Analysis, Why-Why-Because, RCA (5W2H method), 8D Report.
    - **Process Improvement**: PDCA (Plan-Do-Check-Act), Kaizen Suggestion Form, 6 Sigma Project Charter.
    - **Audit & Compliance**: 5S Audit, Gemba Walk Checklist, JSA (Job Safety Analysis), Layered Process Audit (LPA), FAI (First Article Inspection).
    - **Maintenance & Safety**: Daily Machine Check, Non-Conformance Report (NCR), TPM logs, Near Miss Reporting, HAZOP (Hazard and Operability) study.


### 3.2 Dynamic Form Renderer
- **Responsive UI**: Optimized for both Desktop and Mobile-first entry.
- **Real-time Validation**: Client-side feedback based on regex, min/max, and required rules.
- **Section-based Layouts**: Logical grouping of fields into sections and tabs.

---

## 4. Supported Field Types & Schema

| Field Type | Description | Sub-types / Examples |
|:---|:---|:---|
| **Input** | Short and long-form data entry with character constraints. | Text, Textarea, Number, Decimal, Email, URL, Phone, Zip Code |
| **Selection** | Choice based | Dropdown, Multi-select, Radio, Checkbox |
| **Temporal** | Time tracking | Date, DateTime, Time |
| **Media** | Rich data | File Upload (PDF/XLS), Image Capture, Signature |
| **Automated** | System data | QR/Barcode scanning, GPS Location |
| **UI & Action** | Visual & Functional elements | Buttons, Static Labels, Dividers, Rich Text |

### 4.1 Input Field Details
- **Text (Single-line)**: Designed for brief inputs like names, codes, or identifiers. Supports `minLength` and `maxLength` constraints (e.g., max 255 chars) to ensure data consistency.
- **Textarea (Multi-line)**: Ideal for detailed observations, incident reports, or descriptive logs. Allows larger blocks of text with auto-expanding UI for better readability.
- **Number & Decimal**: Optimized for numeric data with support for range validation (`min`/`max`) and step precision (e.g., 0.01 for temperature readings).
- **Specialized Inputs**: 
    - **Email/URL**: Built-in regex validation to ensure correct formatting.
    - **Phone Number**: International code selector with input masking and validation.
    - **Zip Code**: Numeric or alphanumeric entry with country-specific validation rules.
    - **Masked Input**: Pattern-based entry for serial numbers (e.g., `AA-0000-XXX`).

### 4.2 UI & Action Element Details
- **Buttons**: Functional elements that can trigger workflows (e.g., "Submit", "Check Inventory", "Generate PDF"). Supports different styles (Primary, Secondary, Danger).
- **Static Labels & Rich Text**: For providing instructions, warnings (e.g., "Wear PPE before proceeding"), or section headers. Supports Markdown/HTML for rich formatting.
- **Dividers**: Visual lines to separate different sections of a form or report for better clarity.

### Example Rich Component: Search Bar
```json
{
  "id": "search_bar",
  "name": "Search Bar",
  "type": "input",
  "subType": "text",
  "label": "Search",
  "placeholder": "Search assets...",
  "validation": {
    "maxLength": 100,
    "regex": "^[a-zA-Z0-9 _-]*$"
  },
  "events": [
    {
      "event": "onChange",
      "action": "filterList",
      "debounce": 300
    }
  ]
}
```

---

## 5. Validation & Conditional Logic

### 5.1 Validation Rules
- **Structural**: Required, Min/Max Length, Unique Value.
- **Data Integrity**: Min/Max Value, Regex Pattern, Allowed Value lists.

### 5.2 Conditional Logic (`showIf`)
Fields dynamically appear based on user input, enabling complex decision trees without coding.
```json
{
  "showIf": {
    "field": "machineFailure",
    "equals": true
  }
}
```

### 5.3 Dynamic Data Binding (Cascading Filters)
Fields can be linked to external database tables for real-time validation and selection. This supports **Cascading Dropdowns** where one selection filters the options for the next.

**Example Scenario (Place → Plant):**
When a user selects a **Place** (e.g., *Vattamalai*), the **Plant** dropdown is automatically filtered to show only plants associated with that place (e.g., *SMB 2.5MW, VMB 2.5MW*).

**Example JSON Configuration:**
```json
{
  "id": "plant_selector",
  "label": "Select Plant",
  "type": "selection",
  "subType": "dropdown",
  "dataSource": {
    "type": "database",
    "table": "master_plant_data",
    "displayField": "plant_name",
    "valueField": "plant_id",
    "filter": {
      "key": "place_name",
      "matchField": "place_selector" 
    }
  }
}
```

**Sample Master Data (JSON Representation):**
```json
{
  "locations": [
    {
      "place": "Olapalayam",
      "plants": ["SMB 1.0MW", "VMB 1.0MW", "SMB 1.5MW", "VMB 1.5MW"]
    },
    {
      "place": "Vattamalai",
      "plants": ["SMB 2.5MW", "VMB 2.5MW", "SWF 1.0MW", "AWF 1.0MW", "SP 1.0MW"]
    },
    {
      "place": "Udumalpet",
      "plants": ["SMB 4.0MW", "VMF 1.0MW", "SMT 1.0MW"]
    },
    {
      "place": "Vadamadurai",
      "plants": ["VMB 4.0MW", "SSF 1.0MW", "VISHNU 4.0MW"]
    },
    {
      "place": "Dindivanam",
      "plants": ["SWF 2.0MW", "ASWF 2.0MW"]
    }
  ]
}
```

---

## 6. Layout & Canvas Experience (Power Apps Style)

The builder uses a grid-based coordinate system for precise control.

| Feature | Description |
|:---|:---|
| **Grid Overlay** | 12/24-column background for alignment. |
| **Snap-to-Grid** | Components automatically snap to the nearest line. |
| **Layout Metadata** | Stored as row/column/section indices in the JSON schema. |
| **Property Inspector** | Sidebar for editing validation, logic, and permissions. |

---

## 7. Workflow, Permissions & Security

### 7.1 Workflow States
- **Statuses**: Draft → Submitted → Under Review → Approved/Rejected → Closed.
- **Features**: Role-based approvals, comments, and escalation rules.

### 7.2 Role-Based Access Control (RBAC)
- **Roles**: Admin, Manager (Approvals), Operator (Data Entry), Viewer.
- **Levels**: View, Create, Edit, Delete, Approve.

### 7.3 Audit Trail
- **Traceability**: Full tracking of `created_by`, `created_on`, `modified_by`, and `modified_on`.
- **System Metadata**: Device information, IP address, and GPS coordinates for field submissions.
- **Immutability**: Permanent log of "Old Value" vs. "New Value" for every update to a log entry.

---

## 8. Technical & Future Vision

### Technical Stack
- **Frontend**: **Vanilla HTML5, CSS3 (Modern Flex/Grid), and JavaScript (ES6+)**. 
    - *Rationale*: No build steps (Vite/Webpack) required, instant loading, and maximum control over the DOM for the Visual Canvas.
    - *Libraries*: `interact.js` for drag-and-drop/grid-snapping, `Tailwind CSS` (optional CDN) or Custom CSS variables for premium aesthetics.
- **Backend**: FastAPI with JSON-based form metadata storage.
- **Database**: PostgreSQL (JSONB) for flexible schema with relational audit logs.

### Future Enhancements
- **Industrial Connectivity**: BACnet/Modbus integration for live machine data.
- **AI/ML**: Anomaly detection and predictive maintenance alerts.
- **Digital Twin**: Integration with 3D models for visual logging.
- **Voice Logging**: Hands-free data entry for field operators.

---

## 10. Security & Intellectual Property (IP) Protection

### 10.1 Where the "Brain" Lives
The core value of Logit resides in the **Backend (FastAPI)** and **Database (PostgreSQL)**. Even if the frontend HTML/JS/CSS is copied:
- **Zero Functionality**: The copier would lack the API endpoints for form publishing, data validation, and secure storage.
- **No Master Data**: They would not have access to the master database tables (e.g., your Place/Plant data).
- **Audit Failure**: The immutable audit trail and security checks are enforced at the server level.

### 10.2 Frontend Protection Strategies
To deter simple copying and protect the UI logic:
- **Minification & Obfuscation**: Before deployment, the Vanilla JS files will be "obfuscated" (variable names scrambled, white-space removed), making the code unreadable to humans while remaining functional for browsers.
- **JWT Authentication**: All API calls from the frontend require a valid JSON Web Token, ensuring only authorized users can interact with the app's "brains".

---

## 9. Database Architecture & Schema Strategy

To handle dynamic forms without constant database migrations, Logit uses a **Hybrid Relational-JSONB** strategy in PostgreSQL.

### 9.1 Core Tables (ERD Overview)

1.  **`FormDefinitions`**: Stores the "Master" version of each form.
    - `id` (UUID, Primary Key)
    - `name`, `version`, `schema` (JSONB)
    - `is_active` (Boolean) - Enable/Disable forms instantly.
    - **Metadata**: `created_at`, `created_by`, `updated_at`, `updated_by`.

2.  **`LogSubmissions`**: Stores the actual data pushed by operators.
    - `id`, `form_definition_id` (FK), `data` (JSONB)
    - `status` (Workflow Status)
    - **Metadata**: `submitted_at` (created_on), `submitted_by` (created_by), `updated_at`, `updated_by`.
    - `device_metadata` (JSONB) - GPS, OS, Browser Info.

3.  **`MasterLookupData`**: Stores cascading data (like the Place/Plant mapping).
    - `category`, `data` (JSONB)
    - **Metadata**: `updated_at`, `updated_by`.

### 9.2 Why JSONB?
- **Flexibility**: We can add new fields to a form (e.g., adding "Zip Code") without altering the database table structure.
- **Query Performance**: PostgreSQL allows indexing specific keys inside JSONB (e.g., `GIN` indexes), making searches fast even inside the dynamic data.
- **Versioning**: Each submission is linked to a specific `version` of the `FormDefinition`, ensuring that if a form changes, old logs still make sense.

### 9.3 Data Integrity
While the `data` column is flexible, the **Backend Validation Engine** (FastAPI) enforces the rules defined in the `schema` before any write occurs to the database, acting as the "Gatekeeper".

---
*Last Updated: May 6, 2026*
