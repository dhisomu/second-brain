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

*Created: April 18, 2026*
