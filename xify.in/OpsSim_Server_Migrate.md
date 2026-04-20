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
| <a id="TSK-MIG-004"></a>**TSK-MIG-004** | Backend | `backend/main.py` | Implementation of `GET /api/projects/check-name`. Implements [Path 1.1.3](#Path-1.1.3). |
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
| <a id="TC-1.1"></a>**TC 1.1: Unique Name (Happy Path)** | **Arrange**: User in settings modal with unique inputs.<br>**Act**: User enters "Proj_Alpha_01" and clicks Save.<br>**Assert**: HTTP 201 response received; Modal closes; Editor interactive.<br>**Validates**: **[AC 1.1.1](#AC-1.1.1)**, **[AC 1.1.4](#AC-1.1.4)** |
| <a id="TC-1.2"></a>**TC 1.2: Duplicate Name (Unhappy Path)** | **Arrange**: Database contains a project named "Demo".<br>**Act**: User enters "Demo" and attempts to Save.<br>**Assert**: Alert "matching name already exists" shown; Rename/Link buttons appear.<br>**Validates**: **[AC 1.1.2](#AC-1.1.2)**, **[AC 1.1.3](#AC-1.1.3)** |
| <a id="TC-1.3"></a>**TC 1.3: Sanitization (Edge Case)** | **Arrange**: User enters SQL tokens (e.g., `'; DROP TABLE...`).<br>**Act**: User clicks Save.<br>**Assert**: Server processes input as literal string; no database damage.<br>**Validates**: **[AC 1.1.4](#AC-1.1.4)** |

### Flow 2: Existing Project Selection ([Path 1.2](#Path-1.2))
*Tasks: TSK-MIG-011, TSK-MIG-014*

#### Acceptance Criteria
- <a id="AC-1.2.1"></a>**AC 1.2.1**: **GIVEN** the project list page, **WHEN** any existing snapshot is clicked, **THEN** a selection modal must prompt the user to choose between "VIEW" or "EDIT". (Verified by **[TC 2.1](#TC-2.1)**)
- <a id="AC-1.2.2"></a>**AC 1.2.2**: **GIVEN** the selection modal, **WHEN** "VIEW" is selected, **THEN** the browser must navigate to the read-only viewer app. (Verified by **[TC 2.1](#TC-2.1)**)
- <a id="AC-1.2.3"></a>**AC 1.2.3**: **GIVEN** the selection modal, **WHEN** "EDIT" is selected, **THEN** the server must immediately clone the current snapshot to a new version ID and open the editor app. (Verified by **[TC 2.2](#TC-2.2)**)

#### Test Cases (AAA)
| Scenario | Test Logic |
| :--- | :--- |
| <a id="TC-2.1"></a>**TC 2.1: Selection Flow (Happy Path)** | **Arrange**: User clicks an existing thumbnail.<br>**Act**: User clicks "VIEW" button in choice modal.<br>**Assert**: URL changes to `/Apps/xify_opssim_view/` with correct ID.<br>**Validates**: **[AC 1.2.1](#AC-1.2.1)**, **[AC 1.2.2](#AC-1.2.2)** |
| <a id="TC-2.2"></a>**TC 2.2: Version Cloning (Happy Path)** | **Arrange**: User clicks "EDIT" for Version 5 of a project.<br>**Act**: Logic triggers project cloning.<br>**Assert**: Database contains NEW Version 6; Editor opens Version 6.<br>**Validates**: **[AC 1.2.3](#AC-1.2.3)** |
| <a id="TC-2.3"></a>**TC-2.3: Stale Data (Edge Case)** | **Arrange**: Snapshot is manually deleted during user session.<br>**Act**: User clicks "VIEW" for the deleted snapshot.<br>**Assert**: System displays "Snapshot no longer available" error.<br>**Validates**: **[AC 1.2.2](#AC-1.2.2)** |
