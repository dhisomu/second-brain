# Feature Documentation: Serialized Traceability System (RM, WIP, FG)

## 1. User Story
**As a** Industrial Simulation Designer,
**I want** to spawn unique, color-coded, and serialized cubes (Raw Material, Work-In-Progress, Finished Goods),
**So that** I can track every individual part through the production lifecycle with 100% traceability and no identity collisions.

---

## 2. Acceptance Criteria (Gherkin Format)

### **AC-01: Robust Sequential Serialization**
*   **GIVEN** the scene already contains an object named `RM#1`.
*   **WHEN** the user clicks the "Item.RM" button.
*   **THEN** the system must scan the scene and generate the name `RM#2`.
*   **GIVEN** the simulation is reloaded with existing items up to `#10`.
*   **WHEN** a new item is created.
*   **THEN** the counter must automatically resume from `#11`.

### **AC-02: Dynamic Visual Identity**
*   **GIVEN** a new `WIP#5` is created.
*   **WHEN** the object is rendered.
*   **THEN** its surface texture must immediately display "WIP#5" and its color must match the `WIP` schema in `master_data.json`.

### **AC-03: Memory Efficiency (RAM Optimization)**
*   **GIVEN** the user needs to spawn hundreds of serialized parts.
*   **WHEN** objects are created.
*   **THEN** the system must reuse a single shared geometry instance and dispose of unused textures to prevent memory leaks.

---

## 4. Implementation Approach

### **Traceability & Collision Avoidance**
Instead of a simple incrementing integer, the system performs a **Scene-Wide Collision Scan** before name assignment. This ensures that even if items are deleted or imported from external files, the "Next ID" is always truly unique.

### **RAM Optimization (Lightweight Architecture)**
1.  **Shared Geometry**: All 1x1x0.35 cubes share a single `v3d.BoxGeometry` instance in memory.
2.  **Schema-Driven Styling**: Colors and font settings are pulled from `master_data.json` based on the item prefix, reducing hardcoded logic.
3.  **Scoped Counters**: Independent counters for `RM_COUNTER`, `WIP_COUNTER`, and `FG_COUNTER` prevent sequence mixing.

---

## 5. Manual Test Suite

| ID | Test Case | Expected Result |
|:---|:---|:---|
| **TC-01** | **Sequential Creation** | Click RM button 3 times -> Observe `RM#1`, `RM#2`, `RM#3` in scene. |
| **TC-02** | **Deletion Recovery** | Create `RM#1`, `RM#2`. Delete `RM#2`. Click RM button -> System should generate `RM#2` (or skip to `RM#3` based on counter logic) ensuring no duplicate in scene. |
| **TC-03** | **Type Isolation** | Create `RM#1`, then click WIP button -> Observe `WIP#1`. Counters must remain independent. |
| **TC-04** | **Texture Sync** | Duplicate an existing `RM#5` -> Observe the clone is automatically renamed and its texture updated to a unique ID (e.g. `RM#6`). |

---
*Document Created: 2026-04-16*
*Project: OpsSim*
