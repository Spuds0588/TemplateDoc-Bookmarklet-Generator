# Project Name: TemplateDoc Bookmarklet Generator (MVP)

## 1. Product Requirements Document (PRD)

### 1.1 Overview
The TemplateDoc Bookmarklet Generator is a stateless, client-side Single Page Application (SPA). It allows users to build document templates with dynamic variables and compile them into self-contained Javascript bookmarklets. When triggered on any external website, these bookmarklets prompt the user for variable inputs via a lightweight modal, inject a hidden iframe into the host page, and utilize native browser printing to generate a perfectly formatted PDF. 

### 1.2 Target Audience & Use Case
*   **Users:** Mortgage processors, loan officers, and compliance teams.
*   **Use Case:** Generating simple, standardized documents (Letters of Explanation, Asset Verification requests) directly from highly secured, locked-down CRMs without requiring context switching or violating Content Security Policies (CSP) via browser extensions.

### 1.3 Scope (MVP)
**In-Scope:**
*   Wizard SPA with local storage persistence.
*   Left/Right split builder UI (Variables on left, 8.5x11 WYSIWYG editor on right).
*   Variable Types: Text, Long Text, Number, Date, Image (Base64), Signature (Canvas Drawing or Cursive Font).
*   Text replacement variable injection (`{{VariableName}}`).
*   Preview mode before bookmarklet generation.
*   Generated bookmarklet must function entirely independently, using no external CDNs, utilizing native `window.print()` via a hidden iframe.

**Out-of-Scope (Deferred):**
*   Rich text input within the bookmarklet modal.
*   Cloud database syncing (Auth/DB).
*   Chrome Extension deployment.
*   Advanced DOM node variable injection (pill UI in the editor).

---

## 2. Implementation Guide

### 2.1 Tech Stack
*   **Wizard SPA:** HTML5, Vanilla JavaScript (ES6+), Bulma.css (via CDN), Quill.js (via CDN).
*   **Generated Bookmarklet:** Pure Vanilla JS (ES5/ES6), inline vanilla CSS (no external libraries).

### 2.2 Data Model (LocalStorage)
Templates will be saved in `localStorage` as a JSON array under the key `docmagic_templates`.
```json
{
  "id": "1678901234",
  "name": "Letter of Explanation",
  "variables":[
    { "id": "v1", "name": "BorrowerName", "type": "text" },
    { "id": "v2", "name": "Explanation", "type": "longtext" },
    { "id": "v3", "name": "BorrowerSignature", "type": "signature" }
  ],
  "htmlContent": "<p>I, {{BorrowerName}}, am writing to explain...</p>"
}
```

### 2.3 Architecture & Workflows

#### A. Wizard SPA Architecture
1.  **Dashboard:** Reads from `localStorage` to list, edit, or delete existing templates.
2.  **Builder:** 
    *   **Left Panel:** Form to add/remove variables. Generates the `{{VariableName}}` helper text.
    *   **Right Panel:** Quill.js instance. Its container is CSS-styled with a fixed `max-width: 816px` (8.5 inches at 96dpi), minimum height, padding, and box-shadow to simulate a piece of US Letter paper.
3.  **Preview:** Opens a mock version of the bookmarklet's modal. On submit, injects the iframe into the SPA DOM to test printing.
4.  **Compiler:** Takes the `variables` array and `htmlContent`. It uses string interpolation to inject these into a pre-written JavaScript template string (the bookmarklet engine). It then wraps the script in an IIFE, encodes it using `encodeURIComponent`, and prepends `javascript:`.

#### B. The Bookmarklet Execution Engine
The compiled bookmarklet string must execute the following sequence:
1.  **UI Injection:** Create a `<div id="docmagic-overlay">` styled with inline CSS (fixed position, z-index 9999, semi-transparent background). Inside, inject a modal with native HTML form inputs mapped to the template's variables.
2.  **Signature Logic (if applicable):** Map signature variables to a UI containing a `<canvas>` element (with mouse/touch event listeners to draw paths) and a "Type to Sign" text input styled with a system cursive font (`font-family: 'Brush Script MT', cursive;`). Upon saving, extract the canvas data via `canvas.toDataURL()`.
3.  **Compilation:** On form submit, grab values from all inputs. Run a simple regex replace: `templateHTML.replace(/\{\{VariableName\}\}/g, inputValue)`.
4.  **Iframe Injection & Print:**
    *   Create `iframe` (style: `width:0; height:0; border:none; visibility:hidden;`).
    *   Append to `document.body`.
    *   Open `iframe.contentDocument.write()`.
    *   Write standard HTML wrapper, including `@page { margin: 1in; } body { font-family: sans-serif; }` and the compiled HTML payload.
    *   Call `document.close()`.
    *   Wait for load event or short timeout, then call `iframe.contentWindow.print()`.
    *   Destroy iframe and overlay modal after print dialog closes.

---

## 3. Developer Task List

### Phase 1: SPA UI & Scaffolding
- [ ] Initialize `index.html` structure importing Bulma CSS and Quill JS CDNs.
- [ ] Create basic SPA routing/view toggling functions (Dashboard -> Builder -> Preview -> Export).
- [ ] Implement `localStorage` CRUD wrappers for managing templates.
- [ ] Build the Dashboard UI to list saved templates and create a new one.

### Phase 2: Builder Interface
- [ ] Implement the Left Panel: UI to add, edit, and remove variables. Include a "Copy to Clipboard" button for the `{{VariableName}}` syntax.
- [ ] Implement the Right Panel: Initialize Quill.js.
- [ ] Style the Quill editor container to mimic an 8.5x11 sheet of paper with standard margins and box-shadows.
- [ ] Connect auto-saving logic to update `localStorage` seamlessly.

### Phase 3: The Bookmarklet Compiler Engine
- [ ] Write the standalone "Bookmarklet Engine" logic as an isolated JavaScript file (to be stringified).
- [ ] **Engine Task:** Implement the vanilla JS modal generator (overlay, form creation based on passed variables array, inline CSS).
- [ ] **Engine Task:** Implement the Signature block logic natively (canvas drawing implementation and DataURL extraction).
- [ ] **Engine Task:** Implement the hidden iframe creation, DOM writing, `@page` CSS injection, and `window.print()` trigger.
- [ ] Create the Compiler Service in the SPA: Takes template HTML and Variables, injects them into the stringified Engine logic, wraps in an IIFE `javascript:(function(){ ... })();`, and URL encodes it.

### Phase 4: Preview & Polish
- [ ] Build the Preview View: Let the user interact with the generated bookmarklet modal directly inside the SPA before exporting.
- [ ] Build the Export View: Display a large textarea with the raw bookmarklet string, and an anchor tag formatted as a draggable Bookmark link.
- [ ] Add basic field validation to the SPA builder (ensure variable names have no spaces/special characters).
- [ ] Clean up console warnings and ensure iframe garbage collection is handled post-printing.
```

***

**Next Steps:**
If you review this and it looks complete, I can take on the role of the developer and start writing the actual code. I recommend we start by generating the full `index.html` (which will contain the CSS, JS, and HTML in one file for easy testing) tackling Phase 1 and 2. 

Just give me the green light!
