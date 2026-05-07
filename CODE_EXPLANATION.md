# Code Explanation — CRX Integration Tracker

This document explains how the application works, section by section. It's meant to help you (or any developer) understand the code well enough to make changes confidently.

---

## Table of Contents

1. [Overall Architecture](#1-overall-architecture)
2. [Data Layer — How Data is Stored](#2-data-layer--how-data-is-stored)
3. [Application State](#3-application-state)
4. [HTML Structure](#4-html-structure)
5. [CSS — Styling System](#5-css--styling-system)
6. [JavaScript — Function by Function](#6-javascript--function-by-function)
7. [How Views Work](#7-how-views-work)
8. [How Status Colors Work](#8-how-status-colors-work)
9. [How Comments Work](#9-how-comments-work)
10. [How PDF Export Works](#10-how-pdf-export-works)
11. [How to Safely Make Changes](#11-how-to-safely-make-changes)

---

## 1. Overall Architecture

The entire app lives in **one HTML file** with three sections:

```
index.html
├── <style>      → All CSS styling
├── <body>       → All HTML markup (multiple "views" stacked on top of each other)
└── <script>     → All JavaScript logic
```

There is **no server**, **no database**, and **no build process**. The app runs entirely in the browser.

**How multiple "pages" work without a server:**  
Rather than loading different URLs, the app shows and hides different `<div>` blocks using CSS (`display: none` vs `display: block`). These are called *views*. Only one view is visible at a time.

```
View 1: Dashboard (list of all clients)
View 2: Client Detail (individual client page)
```

---

## 2. Data Layer — How Data is Stored

### Storage Location
All data is saved to **`localStorage`** — a built-in browser feature that lets websites store data permanently on your device (survives page refresh, browser close, computer restart).

```javascript
const DB_KEY = 'crx_tracker_v2';
```
This is the key under which all data is stored. Think of it like a named box in your browser.

### The Two Core Functions

```javascript
function loadDB() {
  try { return JSON.parse(localStorage.getItem(DB_KEY)) || { clients: [] }; }
  catch { return { clients: [] }; }
}
```
- Reads the stored data from localStorage
- Parses it from a JSON string back into a JavaScript object
- If nothing is stored yet, returns a blank database: `{ clients: [] }`
- The `try/catch` handles any corruption gracefully

```javascript
function saveDB(db) {
  localStorage.setItem(DB_KEY, JSON.stringify(db));
}
```
- Converts the JavaScript object to a JSON string
- Saves it to localStorage
- **Every action that changes data calls `saveDB()` at the end**

### Data Structure (What Gets Saved)

```json
{
  "clients": [
    {
      "id": "c_1712345678",
      "name": "Geisinger Health",
      "cmmsType": "Maximo",
      "crxConsultant": "John Smith",
      "clientPoc": "Jane Doe",
      "overallStatus": "Active",
      "startDate": "2025-01-15",
      "createdAt": "2025-01-15T10:30:00.000Z",
      "updatedAt": "2025-04-09T14:22:00.000Z",
      "sections": [
        {
          "id": "test_instance",
          "label": "CMMS Test Instance Integration",
          "phases": [
            {
              "id": "test_p1",
              "label": "Phase 1:",
              "tasks": [
                {
                  "id": "t1",
                  "name": "Initial Requirement",
                  "status": "completed",
                  "comments": [
                    {
                      "author": "John Smith",
                      "text": "Requirements doc received from client.",
                      "timestamp": "Jan 20, 2025, 9:15 AM"
                    }
                  ]
                }
              ]
            }
          ]
        }
      ],
      "docs": [
        {
          "type": "SRS",
          "name": "System Requirements Document",
          "link": "https://drive.google.com/...",
          "notes": "Version 2 approved",
          "date": "1/20/2025"
        }
      ],
      "globalComments": [
        {
          "author": "John Smith",
          "text": "Kickoff call scheduled for next week.",
          "timestamp": "Jan 15, 2025, 10:30 AM"
        }
      ]
    }
  ]
}
```

---

## 3. Application State

Three variables track what the app is currently doing:

```javascript
let currentClientId = null;   // Which client is open (null = on dashboard)
let editingClientId = null;   // Which client is being edited in the modal (null = creating new)
let commentContext = null;    // Which task's comment modal is open
                              // Format: { secId, phId, taskId }
```

These are simple global variables. Every function that needs to know "which client are we looking at?" reads `currentClientId`.

---

## 4. HTML Structure

### The Two Main Views

```html
<!-- View 1: Dashboard -->
<div id="viewDashboard" class="view active">
  <div class="dashboard">
    <div class="client-grid" id="clientGrid">
      <!-- Client cards are injected here by JavaScript -->
    </div>
  </div>
</div>

<!-- View 2: Client Detail -->
<div id="viewDetail" class="view">
  <!-- Client info header, tabs, and tab contents -->
</div>
```

The `active` class is what makes a view visible. Only one view has `active` at a time.

### Modals (Popup Dialogs)

There are four modals, all hidden by default:

| Modal ID | Purpose |
|---|---|
| `clientModal` | Add or Edit a client |
| `commentModal` | Per-task comment thread |
| `docModal` | Add a document |
| `confirmOverlay` | Delete confirmation |

Each modal has `.modal-overlay` as its outer wrapper. Clicking outside the modal box closes it:

```javascript
document.querySelectorAll('.modal-overlay').forEach(o =>
  o.addEventListener('click', e => { if (e.target === o) o.classList.remove('open'); })
);
```

### Placeholder Containers

Several elements in the HTML are **empty containers** that JavaScript fills in:

| Element ID | Filled by |
|---|---|
| `clientGrid` | `renderDashboard()` |
| `integrationSections` | `renderSections()` |
| `overviewStats` | `renderStats()` |
| `docsTableBody` | `renderDocs()` |
| `globalChatMessages` | `renderGlobalComments()` |
| `commentThread` | `renderCommentThread()` |

---

## 5. CSS — Styling System

### CSS Variables (Design Tokens)

All colors are defined as CSS custom properties at the top:

```css
:root {
  --bg: #0d0f14;          /* Page background */
  --surface: #151820;     /* Card/panel background */
  --surface2: #1c2030;    /* Slightly lighter panel */
  --surface3: #222840;    /* Table headers, tags */
  --border: #2a3050;      /* Border color */
  --accent: #4f7cff;      /* Primary blue — buttons, links */
  --accent2: #00e5c0;     /* Teal — section titles */
  --text: #e8ecf8;        /* Primary text */
  --text2: #8892b0;       /* Secondary text */
  --text3: #4a5580;       /* Muted/label text */
}
```

**To change the color scheme**, edit these variables — everything updates automatically.

### Status Colors

Each status has a background and text color:

```css
--ns: #3a3f5c;       --ns-text: #8892b0;    /* Not Started — grey */
--ip: #1a3a6a;       --ip-text: #4f9fff;    /* In Progress — blue */
--done: #0d3322;     --done-text: #00e5a0;  /* Completed — green */
--pfc: #3a2a00;      --pfc-text: #ffb830;   /* Pending From Client — amber */
--cbd: #3a1a1a;      --cbd-text: #ff5a5a;   /* Cannot Be Done — red */
--nr: #1a1a3a;       --nr-text: #7878c0;    /* Not Required — purple */
```

### The `.view` Pattern

```css
.view { display: none; }
.view.active { display: block; }
```
Simple show/hide mechanism. JavaScript adds/removes the `active` class.

---

## 6. JavaScript — Function by Function

### Data Functions

| Function | What it does |
|---|---|
| `loadDB()` | Reads all data from localStorage and returns it as an object |
| `saveDB(db)` | Saves the entire data object back to localStorage |
| `defaultSections()` | Returns the template task structure for a new client |

### View / Navigation Functions

| Function | What it does |
|---|---|
| `showView(id)` | Hides all views, shows only the one with the given ID |
| `showDashboard()` | Resets state, renders dashboard, shows dashboard view |
| `showDetail(clientId)` | Sets `currentClientId`, renders all detail sections, shows detail view |

### Render Functions (these build the HTML)

| Function | What it does |
|---|---|
| `renderDashboard()` | Builds all client cards and injects into `#clientGrid` |
| `renderDetail()` | Populates the entire detail page for the current client |
| `renderStats(c)` | Builds the 6 stat boxes (counts per status) |
| `renderSections(c)` | Builds all integration task tables with dropdowns and comment buttons |
| `renderDocs(c)` | Builds the documents table |
| `renderGlobalComments(c)` | Builds the client notes chat feed |
| `renderCommentThread()` | Builds the per-task comment thread inside the modal |

### Action Functions (these change data)

| Function | What it does |
|---|---|
| `saveClient()` | Creates a new client OR saves edits to an existing one |
| `updateTaskStatus(secId, phId, taskId, newStatus)` | Updates a single task's status |
| `saveComment()` | Adds a comment to a specific task |
| `addGlobalComment()` | Adds a note to the client-level notes feed |
| `saveDoc()` | Adds a document entry to the client |
| `deleteDoc(idx)` | Removes a document by its index in the array |
| `deleteCurrentClient()` | Removes the current client entirely from the database |

### Modal Functions

| Function | What it does |
|---|---|
| `openAddClientModal()` | Clears the form and opens the client modal in "create" mode |
| `openEditClientModal()` | Populates the form with current client data, opens in "edit" mode |
| `openCommentModal(secId, phId, taskId, taskName)` | Sets `commentContext`, renders thread, opens modal |
| `openAddDocModal()` | Clears the form and opens the doc modal |
| `openModal(id)` | Generic: adds `.open` class to any modal overlay |
| `closeModal(id)` | Generic: removes `.open` class from any modal overlay |

### Helper / Utility Functions

| Function | What it does |
|---|---|
| `computeStats(client)` | Counts tasks by status, returns totals |
| `statusLabel(s)` | Converts status key to display name (e.g. `'in-progress'` → `'In Progress'`) |
| `statusClass(s)` | Converts status key to CSS class (e.g. `'completed'` → `'s-completed'`) |
| `statusOptions(cur)` | Builds `<option>` HTML for the status dropdown, marking the current value as selected |
| `updateSelectStyle(sel)` | Updates the CSS class on a `<select>` element when the value changes |
| `switchTab(tabId, btn)` | Shows one tab content panel, hides others, updates active button |
| `esc(str)` | Escapes HTML special characters to prevent display bugs |
| `fmtTime()` | Returns current date/time as a formatted string (e.g. "Apr 9, 2025, 2:30 PM") |

---

## 7. How Views Work

When you click a client card on the dashboard, this happens:

```
User clicks card
  → showDetail('c_1712345678') is called
    → currentClientId = 'c_1712345678'
    → renderDetail() runs:
        → loads client from localStorage
        → populates all the text fields (name, cmms, etc.)
        → calls renderStats(), renderSections(), renderDocs(), renderGlobalComments()
    → showView('viewDetail') runs:
        → removes 'active' from viewDashboard
        → adds 'active' to viewDetail
```

When you click "← Dashboard":

```
showDashboard() is called
  → currentClientId = null
  → renderDashboard() re-runs (re-reads latest data from localStorage)
  → showView('viewDashboard')
```

---

## 8. How Status Colors Work

The status `<select>` dropdown uses both a CSS class for color and inline JavaScript to update when changed:

```html
<select class="status-select s-in-progress"
  onchange="updateTaskStatus('test_instance','test_p1','t1', this.value);
            updateSelectStyle(this)">
  <option value="not-started">Not Started</option>
  <option value="in-progress" selected>In Progress</option>
  ...
</select>
```

When the user picks a new status:
1. `updateTaskStatus()` saves the new status to localStorage immediately
2. `updateSelectStyle(this)` swaps the CSS class on the `<select>` to change its color

The CSS classes look like this:

```css
.s-in-progress {
  background: var(--ip);       /* Dark blue background */
  color: var(--ip-text);       /* Bright blue text */
  border-color: rgba(79,159,255,0.3);
}
```

---

## 9. How Comments Work

### Per-Task Comments
Each task object in the data has a `comments` array:

```json
{
  "id": "t1",
  "name": "Initial Requirement",
  "status": "completed",
  "comments": [
    { "author": "John", "text": "Done.", "timestamp": "Apr 9, 2025, 2:00 PM" }
  ]
}
```

When a comment is saved:
1. `commentContext` tells us which `{ secId, phId, taskId }` we're in
2. The code navigates: `client → section → phase → task → comments.push(...)`
3. `saveDB()` persists it
4. `renderCommentThread()` re-renders the modal thread
5. `renderSections()` re-renders the task table so the comment count badge updates

### Global Client Notes
Same pattern, but simpler — stored directly on the client object:

```json
{
  "globalComments": [
    { "author": "John", "text": "Kickoff call done.", "timestamp": "..." }
  ]
}
```

---

## 10. How PDF Export Works

The app loads **jsPDF** from a CDN (a JavaScript library for generating PDFs in the browser):

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
```

The `exportPDF()` function:
1. Creates a new jsPDF document (A4, landscape)
2. Writes the client name and meta info at the top
3. Loops through all sections → phases → tasks, writing each row as text
4. Appends the documents list at the end
5. Calls `doc.save(filename)` which triggers a browser download

The PDF uses only text (no complex table rendering), which keeps the code simple and reliable.

---

## 11. How to Safely Make Changes

### Adding a New Task to All Future Clients
Find `defaultSections()` in the `<script>` block. Add a new entry to the relevant `tasks` array:

```javascript
{ id: 'l7', name: 'Your New Task Name', status: 'not-started', comments: [] }
```

> IDs must be unique across the whole structure. Use a simple pattern like `t7`, `l7`, etc.

### Adding a New Status Option
1. Add it to `STATUS_MAP`:
   ```javascript
   'on-hold': 'On Hold',
   ```
2. Add a CSS class:
   ```css
   .s-on-hold { background: #2a1a3a; color: #c080ff; border-color: rgba(192,128,255,0.3); }
   ```
3. Add the CSS variable colors in `:root` if desired
4. Add it to `statusClass()`:
   ```javascript
   'on-hold': 's-on-hold',
   ```

### Changing the App Title / Branding
Search for `CRX` in the HTML and replace with your preferred name. The logo is in the `<header>`:

```html
<div class="app-logo">CRX <span>/ Integration Tracker</span></div>
```

### Adding a New Field to Clients
1. Add an `<input>` to the client modal HTML
2. In `saveClient()`, read the new input and include it in the client object
3. In `openEditClientModal()`, populate the input with the existing value
4. In `renderDetail()`, display the value somewhere in the detail header

### Updating Without Losing Data
Because data lives in `localStorage` (the browser) and code lives in the HTML file, you can **replace the entire HTML file** and all your data remains intact. The only thing that would break data is if you change `DB_KEY` to a different string — the app would look for data under a new key and find nothing.

---

## Quick Reference — Key IDs

| HTML ID | Purpose |
|---|---|
| `viewDashboard` | Dashboard page container |
| `viewDetail` | Client detail page container |
| `clientGrid` | Where client cards are rendered |
| `integrationSections` | Where task tables are rendered |
| `overviewStats` | Where status count boxes are rendered |
| `docsTableBody` | Where document rows are rendered |
| `globalChatMessages` | Where global notes are rendered |
| `commentThread` | Where per-task comments are rendered |
| `clientModal` | Add/Edit client modal |
| `commentModal` | Task comment modal |
| `docModal` | Add document modal |
| `confirmOverlay` | Delete confirmation overlay |
