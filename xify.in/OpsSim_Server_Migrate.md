# Upcoming Task: Client-to-Server Logic Migration

**Objective**: Move key operations and variables from the client-side (v3d/js) to the server-side (FastAPI/Postgres).

## To-Do List
1. [ ] Audit `visual_logic.js` for hardcoded cost/production variables.
2. [ ] Create `GET /api/config` endpoint to serve these variables dynamically.
3. [ ] Shift simulation "Sum" and "Calculation" logic from JS to Python.
4. [ ] Implement JWT-based validation for all state-changing operations.

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
  - <a id="Path-1.1.5"></a>**AND Path 1.1.5**: Shows the SERVER link to the latest snapshot of the project (**[ID: TSK-MIG-006](#TSK-MIG-006)**)
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
| <a id="TSK-MIG-002"></a>**TSK-MIG-002** | Frontend | `xify_opssim/index.html` | Trigger modal on load if `mode=new` exists. Implements [Path 1.1.1](#Path-1.1.1). |
| <a id="TSK-MIG-004"></a>**TSK-MIG-004** | Backend | `backend/main.py` | [**DONE**](https://github.com/xify-in/prodev-backend/commit/2980cdc) Implementation of `GET /api/projects/check-name`. Implements [Path 1.1.3](#Path-1.1.3). |
| <a id="TSK-MIG-008"></a>**TSK-MIG-008** | Frontend | `xify_opssim/index.html` | "Rename" button listener in Project modal. Implements [Path 1.1.4.1](#Path-1.1.4.1). |
| <a id="TSK-MIG-010"></a>**TSK-MIG-010** | Backend | `backend/main.py` | `POST /api/projects` to save grid & project metadata. Implements [Path 1.1.4.1.2](#Path-1.1.4.1.2). |
| <a id="TSK-MIG-011"></a>**TSK-MIG-011** | Frontend | `opssim_projects.html` | Choice Modal (VIEW vs EDIT) for existing project clicks. Implements [Path 1.2](#Path-1.2). |
| <a id="TSK-MIG-014"></a>**TSK-MIG-014** | Backend | `backend/main.py` | `POST /api/projects/clone` to increment version. Implements [Path 1.2.1.2](#Path-1.2.1.2). |
| <a id="TSK-MIG-015"></a>**TSK-MIG-015** | Frontend | `install.js` / `v_logic.js` | Move Settings Modal/Alert logic to `install.js`. Implements [Path 1.1.4](#Path-1.1.4). |

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
