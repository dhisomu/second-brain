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

**Path 1**: GIVEN the user in `/srv/prodev.xify.in/www/Home/opssim_projects.html`

- **Path 1.1**: WHEN Clicks on "Create Blank Project"
  - **THEN Path 1.1.1**: "/Apps/xify_opssim/index.html" opens with "Project & Grid Settings" window
  - **AND Path 1.1.2**: WHEN user fills "Project & Grid Settings" project Name
  - **AND Path 1.1.3**: THEN the system checks in the SERVER database whether project of same name is available or not
  - **AND Path 1.1.4**: IF available THEN alert the user "A project of matching name already exists. please give unique project Name"
  - **AND Path 1.1.5**: Shows the SERVER link to the latest snapshot of the project
  - **AND Path 1.1.6**: Shows the link to go "opssim_projects.html"

  - **AND Path 1.1.4.1**: WHEN the user click on Rename button
  - **THEN Path 1.1.4.1.1**: Navigate back to "Project & Grid Settings" window with cursor in project name field and old project name in the field selected so that the user recalls the old project name and on typing new project name, it replaces the old project name and checks - **AND Path 1.1.3**:
  - **AND Path 1.1.4.1.2**: If unique project name provided save the project in the database with data from "Project & Grid Settings" Window ( grid color, size etc .,)

- **OR Path 1.2**: WHEN Click on any of the existing project snapshot
  - **THEN Path 1.2.1**: THEN ask the user whether the user want to only VIEW or EDIT the project.
  - **AND Path 1.2.1.1**: IF user want to VIEW the project OPEN the project in viewer app "/Apps/xify_opssim_view/index.html"
  - **THEN Path 1.2.1.2**: IF user want to EDIT the project CLONE the latest snapshot of the project to the next version and OPEN the new version of the project in editor app "/Apps/xify_opssim/index.html" 
  
  

*Created: April 18, 2026*
