# Upcoming Task: Client-to-Server Logic Migration

**Objective**: Move key operations and variables from the client-side (v3d/js) to the server-side (FastAPI/Postgres).

## Overall Migration Progress

### Phase 1: Project Metadata & Security
1. ✅ **TSK-MIG-015 / 010**: Project & Grid Settings Modal (Refactored to `install.js`)
2. ✅ **TSK-MIG-016**: Secure Object Schema Delivery (Server-Gated & Fallback Lookup)
3. ⏳ **TSK-MIG-011**: Project Access Mode: VIEW vs EDIT Selection - **[NEXT]**
4. ⏳ **TSK-MIG-014**: Backend logic for Snapshot Cloning (Version Incrementing)
5. ⏳ **TSK-MIG-017**: Fix Broken Snapshot Link in duplicate name alerts

### Phase 2: Real-Time State Synchronization (Thin Client Execution)
*Goal: Move `objs to save`, `connector info`, and `simdata` to incremental, server-side truth.*
6. ⏳ **TSK-MIG-018**: Incremental Object Property Sync (Real-time updates to server exclusively on object/property change)
7. ⏳ **TSK-MIG-019**: Incremental Connector Logic (Move `connector_info` processing to backend)
8. ⏳ **TSK-MIG-020**: Deprecate Monolithic Client-Side Save (Remove full state assembly from frontend)

### Phase 3: Security & Session Hardening
*Goal: Prevent session hijacking and enforce strict access controls.*
9. ⏳ **TSK-MIG-021**: Reduce Session Expiry (Drop `max_age` from 7 days to 24 hours in Main Backend)
10. ⏳ **TSK-MIG-022**: Implement User-Agent Fingerprinting (Bind session token to browser string; invalidate if changed)

> [!NOTE]
> **Why not track MAC or IP Addresses?**
> *   **MAC Addresses**: Operate at Layer 2 (hardware) and are stripped by the first router. A web server cannot see a client's true MAC address unless a native desktop agent is installed, which breaks the "Thin Client" philosophy.
> *   **IP Addresses**: Binding sessions strictly to IPs is dangerous for modern web apps. Users on mobile hotspots, corporate VPNs, or satellite ISPs often have dynamic IPs that change mid-session, which would result in aggressive false-positive logouts.
> *   **Solution**: Browser Fingerprinting (User-Agent) + Rolling Tokens (Shorter Expiry) is the industry standard.

## Why this matters?
- **Security**: Prevents users from modifying cost/timing variables in the browser console.
- **Consistency**: Centralizes the "Source of Truth" in the database.
- **Performance**: Keeps the client-side lightweight.

## Functional Requirements (Project Creation Workflow)

<a id="Path-1"></a>**Path 1**: GIVEN the user in `/srv/prodev.xify.in/www/Home/opssim_projects.html` (**[ID: TSK-MIG-000](#TSK-MIG-000)**)

- <a id="Path-1.1"></a>**Path 1.1**: WHEN Clicks on "Create Blank Project" (**[ID: TSK-MIG-001](#TSK-MIG-001)**)
  - <a id="Path-1.1.1"></a>**THEN Path 1.1.1**: "/Apps/xify_opssim/index.html" opens with "Project & Grid Settings" window (**[ID: TSK-MIG-002](#TSK-MIG-002)**)
  - <a id="Path-1.1.2"></a>**AND Path 1.1.2**: WHEN user fills "Project & Grid Settings" project Name (**[ID: TSK-MIG-003](#TSK-MIG-003)**)
  - <a id="Path-1.1.3"></a>**AND Path 1.1.3**: THEN the system checks in the SERVER database whether project of same name is available or not (**[ID: TSK-MIG-004](#TSK-MIG-004)**)
  - <a id="Path-1.1.4"></a>**AND Path 1.1.4**: IF available THEN alert the user "A project of matching name already exists. please give unique project Name" (**[ID: TSK-MIG-015](#TSK-MIG-015)**)
  - <a id="Path-1.1.5"></a>**AND Path 1.1.5**: Shows the SERVER link to the latest snapshot of the project (**[ID: TSK-MIG-017](#TSK-MIG-017)**)
  - <a id="Path-1.1.6"></a>**AND Path 1.1.6**: Shows the link to go "opssim_projects.html" (**[ID: TSK-MIG-007](#TSK-MIG-007)**)

  - <a id="Path-1.1.4.1"></a>**AND Path 1.1.4.1**: WHEN the user click on Rename button (**[ID: TSK-MIG-008](#TSK-MIG-008)**)
    - <a id="Path-1.1.4.1.1"></a>**THEN Path 1.1.4.1.1**: Navigate back to "Project & Grid Settings" window with cursor in project name field... (**[ID: TSK-MIG-009](#TSK-MIG-009)**)
    - <a id="Path-1.1.4.1.2"></a>**AND Path 1.1.4.1.2**: If unique project name provided save project to DB with grid data... (**[ID: TSK-MIG-010](#TSK-MIG-010)**)

- <a id="Path-1.2"></a>**OR Path 1.2**: WHEN Click on any of the existing project snapshot (**[ID: TSK-MIG-011](#TSK-MIG-011)**)
  - <a id="Path-1.2.1"></a>**THEN Path 1.2.1**: THEN ask the user whether the user want to only VIEW or EDIT the project. (**[ID: TSK-MIG-012](#TSK-MIG-012)**)
  - <a id="Path-1.2.1.1"></a>**AND Path 1.2.1.1**: IF user want to VIEW user OPEN project in viewer app (**[ID: TSK-MIG-013](#TSK-MIG-013)**)
  - <a id="Path-1.2.1.2"></a>**THEN Path 1.2.1.2**: IF user want to EDIT CLONE snapshot and OPEN in editor (**[ID: TSK-MIG-014](#TSK-MIG-014)**) 
  
  

*Created: April 18, 2026*

## Detailed Implementation Plan (Task Breakdown)

| Task ID | Component | Filename | Change Description |
| :--- | :--- | :--- | :--- |
| <a id="TSK-MIG-001"></a>**TSK-MIG-001** | Frontend | `opssim_projects.html` | Listen for "Create Blank Project" click. Implements [Path 1.1](#Path-1.1). |
| <a id="TSK-MIG-002"></a>**TSK-MIG-002** | Frontend | `xify_opssim/install.js` | [**IN REVIEW**] (Repo: xify.in, Branch: proDev, Commit: [79aaf4a](https://github.com/xify-in/xify.in/commit/79aaf4a)) Modern modal trigger on load. Implements [Path 1.1.1](#Path-1.1.1). |
| <a id="TSK-MIG-004"></a>**TSK-MIG-004** | Backend | `backend/main.py` | [**IN REVIEW**] (Repo: xify.in, Branch: proDev, Commit: [2980cdc](https://github.com/xify-in/xify.in/commit/2980cdc)) Implementation of `GET /api/projects/check-name`. Implements [Path 1.1.3](#Path-1.1.3). |
| <a id="TSK-MIG-008"></a>**TSK-MIG-008** | Frontend | `install.js` | [**IN REVIEW**] (Repo: xify.in, Branch: proDev, Commit: [79aaf4a](https://github.com/xify-in/xify.in/commit/79aaf4a)) Integrated into the modern settings modal logic. Implements [Path 1.1.4.1](#Path-1.1.4.1). |
| <a id="TSK-MIG-010"></a>**TSK-MIG-010** | Backend | `backend/main.py` | [**IN REVIEW**] (Repo: xify.in, Branch: proDev, Commit: [2c59468](https://github.com/xify-in/xify.in/commit/2c59468)) `POST /api/projects` upsert for grid & project metadata. Implements [Path 1.1.4.1.2](#Path-1.1.4.1.2). |
| <a id="TSK-MIG-011"></a>**TSK-MIG-011** | Frontend | `opssim_projects.html` | Project Access Mode: VIEW vs EDIT Selection for existing project clicks. Implements [Path 1.2](#Path-1.2). |
| <a id="TSK-MIG-014"></a>**TSK-MIG-014** | Backend | `backend/main.py` | `POST /api/projects/clone` to increment version. Implements [Path 1.2.1.2](#Path-1.2.1.2). |
| <a id="TSK-MIG-015"></a>**TSK-MIG-015** | Frontend | `install.js` | [**IN REVIEW**] (Repo: xify.in, Branch: proDev, Commit: [79aaf4a](https://github.com/xify-in/xify.in/commit/79aaf4a)) Refactored Settings Modal to `install.js` with server-side name check interceptor. Implements [Path 1.1.4](#Path-1.1.4). |
| <a id="TSK-MIG-016"></a>**TSK-MIG-016** | Fullstack | `backend/main.py` & `install.js` | [**IN REVIEW**] (Repo: xify.in, Branch: proDev, Commit: [625027b](https://github.com/xify-in/xify.in/commit/625027b)) Server-gated object schema delivery via `/api/objects/schema`. Includes **v3d_prefix fallback** to support property window lookups by scene name. |
| <a id="TSK-MIG-017"></a>**TSK-MIG-017** | Frontend | `install.js` | Fix Broken "View Latest Snapshot" link — must use `project_name` and `version` params instead of raw JSON path. Implements [Path 1.1.5](#Path-1.1.5). |
| <a id="TSK-MIG-018"></a>**TSK-MIG-018** | Fullstack | `backend/main.py` & `visual_logic.js` | Send `simdata`/`ObjProp` updates directly to a new `PUT /api/objects/{id}` endpoint on value change, removing reliance on `objs to save` string buildup. |
| <a id="TSK-MIG-019"></a>**TSK-MIG-019** | Fullstack | `backend/main.py` & `visual_logic.js` | Sync `connector_info` immediately to a new `POST/DELETE /api/connectors` endpoint when a connection is formed or broken. |
| <a id="TSK-MIG-020"></a>**TSK-MIG-020** | Frontend | `visual_logic.js` | Remove monolithic `"cloudSave?"` logic. Replacing it with a minimal signaling call `POST /api/projects/snapshot` where the backend assembles the final JSON state from its existing tracked data. |

## Acceptance Criteria (Given-When-Then) & Test Cases (Arrange-Act-Assert)

### Flow 1: New Project Creation ([Path 1.1](#Path-1.1))
*Tasks: TSK-MIG-001, 002, 004, 015, 010*

#### Acceptance Criteria
- <a id="AC-1.1.1"></a>**AC 1.1.1**: **GIVEN** a user on `opssim_projects.html`, **WHEN** they click "Create Blank Project", **THEN** they are directed to the editor and the "Project & Grid Settings" modal must appear immediately. (Verified by **[TC 1.1](#TC-1.1)**)
- <a id="AC-1.1.2"></a>**AC 1.1.2**: **GIVEN** the settings modal is open, **WHEN** a user enters a project name that already exists in the database, **THEN** the system must block the save action and display a duplicate name alert. (Verified by **[TC 1.2](#TC-1.2)**)
- <a id="AC-1.1.3"></a>**AC 1.1.3**: **GIVEN** a duplicate name alert is active, **WHEN** the user clicks "Rename", **THEN** the focus must return to the name field with the existing text fully selected for overwrite. (Verified by **[TC 1.2](#TC-1.2)**)
- <a id="AC-1.1.4"></a>**AC 1.1.4**: **GIVEN** a unique project name and valid settings, **WHEN** the user clicks "Save", **THEN** the project metadata must be persisted to the PostgreSQL database before the simulation scene initializes. (Verified by **[TC 1.1](#TC-1.1)**)

#### Test Cases (AAA)
| Scenario | Test Logic |
| :--- | :--- |
| <a id="TC-1.1"></a> [TC 1.1: Unique Name (Happy Path)](#TC-1.1-spec) | **Arrange**: User in `"Project & Grid Settings"` window with unique inputs.<br>**Act**: User enters "Proj_Alpha_01" and clicks Save.<br>**Assert**: The settings window disappears; the 3D scene (Editor) becomes interactive.<br>**Validates**: **[AC 1.1.1](#AC-1.1.1)**, **[AC 1.1.4](#AC-1.1.4)** |
| <a id="TC-1.2"></a> [TC 1.2: Duplicate Name (Unhappy Path)](#TC-1.2-spec) | **Arrange**: Database contains a project named "Demo".<br>**Act**: User enters "Demo" and attempts to Save.<br>**Assert**: A window pops up with message "matching name already exists"; Rename button appears.<br>**Validates**: **[AC 1.1.2](#AC-1.1.2)**, **[AC 1.1.3](#AC-1.1.3)** |
| <a id="TC-1.3"></a> [TC 1.3: Sanitization (Edge Case)](#TC-1.3-spec) | **Arrange**: User enters SQL tokens (e.g., `'; DROP TABLE...`).<br>**Act**: User clicks Save.<br>**Assert**: Project is saved successfully; Name shown exactly as typed in the project list.<br>**Validates**: **[AC 1.1.4](#AC-1.1.4)** |

### Flow 2: Existing Project Selection ([Path 1.2](#Path-1.2))
*Tasks: TSK-MIG-011, TSK-MIG-014*

#### Acceptance Criteria
- <a id="AC-1.2.1"></a>**AC 1.2.1**: **GIVEN** the project list page, **WHEN** any existing snapshot is clicked, **THEN** a selection modal must prompt the user to choose between "VIEW" or "EDIT". (Verified by **[TC 2.1](#TC-2.1)**)
- <a id="AC-1.2.2"></a>**AC 1.2.2**: **GIVEN** the selection modal, **WHEN** "VIEW" is selected, **THEN** the browser must navigate to the read-only viewer app. (Verified by **[TC 2.1](#TC-2.1)**)
- <a id="AC-1.2.3"></a>**AC 1.2.3**: **GIVEN** the selection modal, **WHEN** "EDIT" is selected, **THEN** the server must immediately clone the current snapshot to a new version ID and open the editor app. (Verified by **[TC 2.2](#TC-2.2)**)

#### Test Cases (AAA)
| Scenario | Test Logic |
| :--- | :--- |
| <a id="TC-2.1"></a> [TC 2.1: Selection Flow (Happy Path)](#TC-2.1-spec) | **Arrange**: User clicks an existing thumbnail.<br>**Act**: User clicks "VIEW" button in the selection window.<br>**Assert**: Screen transitions to Viewer app; Editing/Settings tools are removed from UI.<br>**Validates**: **[AC 1.2.1](#AC-1.2.1)**, **[AC 1.2.2](#AC-1.2.2)** |
| <a id="TC-2.2"></a> [TC 2.2: Version Cloning (Happy Path)](#TC-2.2-spec) | **Arrange**: User clicks "EDIT" for Version 5 of a project.<br>**Act**: Logic triggers project cloning.<br>**Assert**: Editor opens; Version 6 is visible in the page header label.<br>**Validates**: **[AC 1.2.3](#AC-1.2.3)** |
| <a id="TC-2.3"></a> [TC 2.3: Stale Data (Edge Case)](#TC-2.3-spec) | **Arrange**: Snapshot is manually deleted during user session.<br>**Act**: User clicks "VIEW" for the deleted snapshot.<br>**Assert**: Window displays "Snapshot no longer available" with a Return button.<br>**Validates**: **[AC 1.2.2](#AC-1.2.2)** |
## Detailed Test Case Specifications (AAA Reference)

### <a id="TC-1.1-spec"></a>[TC 1.1: Unique Name (Happy Path)](#TC-1.1)
- **Scenario:** The user provides a project name that does not exist in the database.
- **Arrange:** 
    - Application is in the Editor mode (`index.html`).
    - `"Project & Grid Settings"` window is open.
    - "NewProject_01" does not exist in the database.
- **Act:** 
    - User types "NewProject_01" in the name field.
    - User clicks the "Save" button.
- **Assert:** 
    - The `"Project & Grid Settings"` window closes.
    - The 3D scene (grid/environment) becomes active/clickable.
    - No warning or error windows are visible.
- **Validates:** **[AC 1.1.1](#AC-1.1.1)**, **[AC 1.1.4](#AC-1.1.4)**

---

### <a id="TC-1.2-spec"></a>[TC 1.2: Duplicate Name (Unhappy Path)](#TC-1.2)
- **Scenario:** The user provides a project name that already exists in the database.
- **Arrange:** 
    - A project named "ExistingProject" already exists.
    - `"Project & Grid Settings"` window is open.
- **Act:** 
    - User types "ExistingProject" and clicks "Save".
- **Assert:** 
    - A popup alert titled `"Warning"` (or similar) appears.
    - Message "A project of matching name already exists" is visible.
    - A button labeled `"Rename"` is shown.
    - Clicking `"Rename"` moves the typing cursor into the name field and highlights the text.
- **Validates:** **[AC 1.1.2](#AC-1.1.2)**, **[AC 1.1.3](#AC-1.1.3)**

---

### <a id="TC-1.3-spec"></a>[TC 1.3: Sanitization (Edge Case)](#TC-1.3)
- **Scenario:** The user attempts to inject SQL commands into the project name field.
- **Arrange:** 
    - `"Project & Grid Settings"` window is open.
- **Act:** 
    - User enters `'; DROP TABLE projects; --` in the name field and clicks "Save".
- **Act:** 
    - System treats the string as a literal project name.
- **Assert:** 
    - The project is saved successfully without crashing.
    - The user can see the exact string (with symbols) as the Project Name in the dashboard list.
    - No database corruption or site failure occurs.
- **Validates:** **[AC 1.1.4](#AC-1.1.4)**

---

### <a id="TC-2.1-spec"></a>[TC 2.1: Selection Flow (Happy Path)](#TC-2.1)
- **Scenario:** The user wants to view an existing project in read-only mode.
- **Arrange:** 
    - User is on the `opssim_projects.html` dashboard.
    - A list of existing project snapshots is displayed.
- **Act:** 
    - User clicks on a project thumbnail.
    - User selects `"VIEW"` from the choice window.
- **Assert:** 
    - The screen transitions to the Viewer application.
    - User confirms that the `"Grid Settings"` and editing tools are missing from the menu.
    - The model is present for viewing only.
- **Validates:** **[AC 1.2.1](#AC-1.2.1)**, **[AC 1.2.2](#AC-1.2.2)**

---

### <a id="TC-2.2-spec"></a>[TC 2.2: Version Cloning (Happy Path)](#TC-2.2)
- **Scenario:** The user wants to edit an existing project snapshot.
- **Arrange:** 
    - Project "Project_X" is at Snapshot 5.
- **Act:** 
    - User clicks the thumbnail and selects "EDIT".
- **Assert:** 
    - The Editor application opens successfully.
    - The user sees that the "Version ID" or "Snapshot ID" in the top header has increased by 1 from the original.
    - All objects from the original snapshot are present in the new version.
- **Validates:** **[AC 1.2.3](#AC-1.2.3)**

---

### <a id="TC-2.3-spec"></a>[TC-2.3: Stale Data (Edge Case)](#TC-2.3)
- **Scenario:** The selected snapshot is removed while the user is browsing.
- **Arrange:** 
    - Dashboard is open with Snapshot 10 visible.
    - Another process/admin deletes Snapshot 10 from the database.
- **Act:** 
    - User clicks on the thumbnail for Snapshot 10.
- **Assert:** 
    - The selection window pops up, but after choosing an option, a `"Snapshot no longer available"` message appears.
    - A `"Return to Dashboard"` button is provided to the user.
- **Validates:** **[AC 1.2.2](#AC-1.2.2)**
# Technical Deep-Dive: Project & Grid Settings Logic

This document details the mechanics of the "Project & Grid Settings" module in OpsSim as it currently exists within the Verge3D-generated logic, and outlines the plan for server-side migration.

## 1. Grid Configuration Popup
**Location**: `visual_logic.js` (approx. lines 987 - 1210)

This is a Vanilla JavaScript DOM injection that creates the initial workspace setup window.

*   **HTML Structure**: Generated dynamically as a `gridConfigModal` overlay.
*   **Fields**:
    *   `Project Name`: Required field for identifying the workspace.
    *   `Dimensions (X, Y)`: Numerical inputs for plan size.
    *   `Plane Color`: Hex color picker for the floor material.
*   **Prefill Logic**: On open, it checks the global `window` object (`projectName`, `gridSizeX`, etc.) to restore previous state.
*   **Global Variables**: It reads/writes to `window.projectName`, `window.gridSizeX`, `window.gridSizeZ`, and `window.gridcolor`.
*   **Trigger Mechanism**: If `project_name` is absent from the URL on load, `window.openGridConfigPopup()` is triggered automatically.

## 2. Submission Handler (AfterGridDimSubmitExecute)
**Location**: `visual_logic.js` (approx. lines 4523 - 4581)

This logic acts as the bridge between the HTML UI and the 3D Scene.

*   **Callback**: `window.onGridConfigSubmit(data)`
*   **Workflow**:
    1.  **Validation**: Validates that dimensions are positive numbers.
    2.  **Cleanup**: Locates the object named `GridPlane` in the scene and removes it to avoid overlapping floors.
    3.  **Reconstruction**: Triggers `window.createGrid()` with the new validated parameters.
    4.  **URL Sync**: Invokes `updateProjectURL(projectName)` which updates the browser's address bar using `history.replaceState`. This ensures the project name persists through page refreshes.
    5.  **HUD Update**: Notifies the on-screen Grid Info HUD to refresh its display stats.

## 3. Grid Creation Logic (createGridFunction)
**Location**: `visual_logic.js` (approx. lines 4583 - 4689)

The core 3D generation routine that builds the physical floor.

*   **Material**: Uses `MeshStandardMaterial` with `roughness: 1.0` to ensure a matte, industrial look that showcases shadows clearly.
*   **Geometry**: Creates a `PlaneGeometry` scaled to the user's inputs.
*   **Tiling System**:
    *   Loads `media/grid.png`.
    *   Calculates `repeatX = gridSizeX / 10` and `repeatY = gridSizeY / 10`.
    *   **Effect**: This ensures the grid cells always represent exactly 10 meters, regardless of the room size.
*   **Properties**: Sets `receiveShadow = true` and labels the mesh `GridPlane` for future lookups.

---

## 4. Migration & Integration Plan

To formalize the server-gated architecture (TSK-MIG-015), the logic is being transitioned as follows:

### Step 1: Interception (Logic Hijack)
Move the popup HTML and validation logic from `visual_logic.js` to `install.js`. By using `Object.defineProperty` on `window.openGridConfigPopup`, we prevent the Verge3D version from ever showing.

### Step 2: Server-Side Validation
Enhance the "Project Name" field with an `oninput` listener that calls the backend `check-name` endpoint.
*   **Collision Action**: If name exists, show the "Latest Snapshot" link and block the "Apply" button.

### Step 3: Master Record Persistence (TSK-MIG-010)
Update the `onGridConfigSubmit` logic to perform a `POST /api/projects` call. 
*   **Goal**: Ensure the server has a record of the project dimensions and color BEFORE the grid is even created in 3D.
*   **Benefit**: This allows non-editor users (like the View App) to pull accurate grid settings directly from the database master record.

### Step 4: Traceability Mapping
Ensure all grid changes are linked to the migration document for strict bi-directional traceability between BDD criteria and the implementation in `install.js`.

## Architecture Deep Dive: Server-Gated Object Schema (TSK-MIG-016)

This section documents the transition of object property logic from static file delivery to dynamic server-side authorization.

### BEFORE (Static JSON Vulnerability)
*   **Logic Container**: `model_data.json` in the `/Apps/xify_opssim/` directory.
*   **Mechanism**: `fetch("model_data.json")` relative call in `install.js`.
*   **Failure Point**: Nginx protected the `/Apps/` folder with `auth_request`. While the main app (HTML) was allowed, secondary fetch calls to `.json` files were being intercepted and redirected (`302`) to the login page (`/index.html`), causing the "Failed to load schema" alert.
*   **Security Risk**: Object configurations (industrial logic, property names) were stored as plain text files available to any browser that could bypass the folder redirect.

### AFTER (Thin Client / API Delivery)
*   **Endpoint**: `GET /api/objects/schema/{objectId}`
*   **Backend Logic**: FastAPI reads the schema from a non-public `backend/model_data.json` file.
*   **Session Gating**: The API requires a valid `session_id` cookie. Requests without authentication are rejected with `401 Unauthorized`.
*   **Fallback Resolution**: The API supports dual-mode lookups:
    1.  **Direct Match**: Locates schema by Button ID (e.g., `rmItemBtn`) used during initial addition.
    2.  **Scene Match**: Performs reverse lookup by `v3d_prefix` (e.g., `Cube.RM`). This ensures the property window can load for existing objects in the 3D scene without the frontend needing to know the original button ID.
*   **Protocol**: `install.js` uses `{ credentials: "include" }` on the fetch call to ensure session cookies are passed through same-origin security filters.
*   **Granularity**: Instead of downloading the entire 7KB+ dictionary of all possible objects, the client only receives the specific JSON segment for the object being accessed, keeping the frontend "meatless" and secure.

## Session Summary (April 20, 2026)

### Achievements
*   **TSK-MIG-010**: Successfully implemented server-side project & grid metadata persistence.
*   **TSK-MIG-015**: Refactored the Project Settings Modal to a modern UI in `install.js` and implemented auto-grid initialization for blank workspaces.
*   **TSK-MIG-016**: Implemented secure, session-gated object schema delivery via a backend API (`/objects/schema`), resolving Nginx static file access issues.
*   **Legacy Cleanup**: Manually removed legacy UI code from `visual_logic.js` and modernized the environment configuration logic.

### Current State
*   **Flow 1 (New Project Creation)**: Fully implemented and **Ready for Testing**.
*   **Thin Client Stability**: The application is now significantly more "meatless," with the server acting as the authoritative source for project names, grid settings, and object schemas.

### Next Steps (for tomorrow)
1.  **Formal Review**: User to confirm Flow 1 status based on defined test cases.
2.  **TSK-MIG-011**: Implement the **Choice Modal** (VIEW vs EDIT) on the Project Dashboard.
3.  **TSK-MIG-014**: Implement the **Snapshot Cloning** logic on the backend.

**Commit Traceability**: End-of-session push [a67982e](https://github.com/xify-in/xify.in/commit/a67982e)
