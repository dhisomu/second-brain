# Feature Documentation: Dynamic Data-Driven Texture System (OpsSim)

## 1. User Story
**As a** Simulation Designer/Administrator,
**I want** to manage 3D object surface properties (text, font, colors, etc.) through an external configuration file (`master_data.json`),
**So that** I can add new styling parameters or update default behaviors without requiring a developer to modify the application's core JavaScript code.

---

## 2. Acceptance Criteria (Gherkin Format)

### **AC-01: Dynamic UI Generation**
*   **GIVEN** a `master_data.json` file contains a defined `textTexture` schema.
*   **WHEN** the user opens the "Object Property" window.
*   **THEN** the system must render input fields automatically, using **dropdown selects** for alignment and **number inputs** for padding/size.

### **AC-02: Smart Data Sanitization (Import & Sync)**
*   **GIVEN** an object has a missing or `"undefined"` alias (common in legacy imports).
*   **WHEN** the object is initialized or the property window opens.
*   **THEN** the system must automatically restore the alias to match the object's scene name.
*   **GIVEN** imported data contains text styling but the "Enabled" toggle is missing.
*   **THEN** the system must auto-enable the texture so labels appear immediately.

### **AC-02: Automated Font Scaling**
*   **GIVEN** the `fontSize` parameter is set to `"auto"`.
*   **WHEN** the surface texture is generated.
*   **THEN** the system must calculate the optimal size to fit the text string within the surface bounds.

### **AC-03: Unified Palette & Custom Colors**
*   **GIVEN** the user interacts with any color field in the Propeties window.
*   **WHEN** they click a preset swatch or the **"+" custom picker**.
*   **THEN** the system must apply the color to the object and sync the Main Color with the Texture Background Color (`bgColor`).

### **AC-04: Optional Texture Toggle**
*   **GIVEN** the "Enable Text Texture" toggle is present.
*   **WHEN** the user checks or unchecks the toggle.
*   **THEN** the system must either generate the canvas texture or remove it and restore the original material color instantly.

### **AC-05: Automatic Instance Identification**
*   **GIVEN** a new object is created or an existing object lacks text.
*   **WHEN** initialized.
*   **THEN** the surface text must default to the unique object name (e.g., "Cube#10") instead of placeholder text like "INPUT".

---

## 3. Manual Test Suite (AAA Framework)

### **A. Happy Paths (Normal Usage)**
*   **TC-01: Unified Color Sync**: Change object color in "General" tab; assert "Text Style" bgColor and 3D texture update instantly.
*   **TC-02: Auto-Font Fitting**: Enter long text (e.g., "ZONE-A-01") with `fontSize: auto`; assert text scales down to fit the face exactly.
*   **TC-03: Dropdown Alignment**: Select "Top" and "Left" from dropdowns; assert text moves to the top-left corner of the cube face instantly.
*   **TC-04: Sequential Creation**: Spawn a new Cube (Cube#12); assert Text Texture is automatically set to "Cube#12".

### **B. Unhappy Paths (Edge Cases & Resilience)**
*   **TC-05: The "Undefined" Alias Fix**: Import a JSON file where `alias: "undefined"` or `alias: ""`; assert the UI and 3D label display the actual object name (e.g., "Cube#9") instead of the literal word "undefined".
*   **TC-06: Empty Text Fallback**: Clear the "Surface Text" input completely; assert the 3D texture falls back to the Object Name instead of becoming an invisible/empty face.
*   **TC-07: Invalid Numeric Inputs**: Enter text or negative numbers into the "Padding" field; assert the system sanitizes the input to a valid number (e.g., 0) without crashing the canvas renderer.
*   **TC-08: Network Asset Failure**: Temporarily rename `master_data.json` to simulate a 404 error; assert the property window still opens using the "Triple-Safety" hardcoded defaults without any console errors.
*   **TC-09: Auto-Enable on Import**: Import a JSON that has text data but `textTexture: false` or missing; assert the system "Smart-Enables" it so the user doesn't have to manually toggle it for labels to appear.

---

## 4. Implementation Details

### **Unified Palette Architecture**
The system now uses a shared palette component for all color fields. The `bgColor` (Background Color) of the text texture is hard-linked to the main `simData.color` of the object to ensure visual consistency.

### **Priority Flow (Waterfall)**
1.  **Saved State**: Object-specific `simData` overrides.
2.  **Sanitized master Data**: `master_data.json` values (normalized for booleans/nulls).
3.  **Hardcoded Safety**: Internal defaults in `install.js`.

### **Error Resilience (Triple-Safety)**
The system implements a "Safe Navigation" architecture. If the `master_data.json` fails to load or contains partial data, the system automatically falls back to hardcoded defaults for each specific field, preventing `TypeErrors` and ensuring the app remains 100% stable during network instability.

### **Modified Files**
*   [`install.js`](file:///srv/dev.xify.in/www/Home/Apps/xify_opssim/install.js): Unified Palette, Custom Picker, Toggle Logic, and Schema Sync.
*   [`master_data.json`](file:///srv/dev.xify.in/www/Home/Apps/xify_opssim/master_data.json): Added `textTexture` enabled/disabled states.

---
*Document Updated: 2026-04-13*
*Project: OpsSim*
