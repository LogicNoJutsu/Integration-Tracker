# CRx Integration Tracker

A lightweight, single-file web application for tracking CMMS integration statuses across multiple clients — no backend, no database, no dependencies to install.

> Built for CRx consultants to manage client integration workflows, document links, and team notes in one place.

---

## 🚀 Live Demo / Hosting on GitHub Pages

This app is a **single HTML file** — host it for free on GitHub Pages in under 5 minutes:

### Step 1 — Create a GitHub Repository
1. Go to [github.com](https://github.com) and sign in
2. Click **"New repository"**
3. Name it something like `CRx-integration-tracker`
4. Set it to **Public** (required for free GitHub Pages)
5. Click **"Create repository"**

### Step 2 — Upload the File
1. In your new repo, click **"Add file" → "Upload files"**
2. Drag and drop `integration-tracker.html`
3. **Important:** Rename it to `index.html` before uploading (or rename it locally first)
4. Click **"Commit changes"**

### Step 3 — Enable GitHub Pages
1. Go to your repo's **Settings** tab
2. Scroll down to **"Pages"** in the left sidebar
3. Under **"Branch"**, select `main` and folder `/root`
4. Click **Save**
5. Wait ~60 seconds, then your app will be live at:
   ```
   https://YOUR-USERNAME.github.io/CRx-integration-tracker/
   ```

### Step 4 — Updating the App Later
Whenever you get an updated `index.html`:
1. Go to your repo on GitHub
2. Click on `index.html`
3. Click the **pencil (edit) icon** or **"Upload a new version"**
4. Commit the change — the live site updates automatically

> ⚠️ **Your data is safe across updates.** All client data is stored in your browser's `localStorage`, completely separate from the HTML file itself. Replacing the HTML file never touches your data.

---

## ✨ Features

| Feature | Description |
|---|---|
| **Multi-client Dashboard** | Landing page showing all integrations with progress bars and status counts |
| **Client Profiles** | Store Client Name, CMMS Type, CRx Consultant, Client POC, and Overall Status |
| **Integration Task Table** | Full phase-by-phase task tracking (Test Instance + Live Instance) |
| **6 Status Types** | Not Started · In Progress · Completed · Pending From Client · Cannot Be Done · Not Required |
| **Per-task Comments** | Threaded comment log per task with author name and timestamp |
| **Client Notes Tab** | Global chat-style notes log for each client |
| **Document Library** | Store and link client documents with type, name, URL, and date |
| **PDF Export** | Download a formatted status report for any client |
| **Persistent Storage** | All data saved to browser localStorage — survives page refreshes and updates |
| **No Install Required** | Single HTML file, works in any modern browser |

---

## 📋 How to Use

### Creating a New Client
1. Click **"+ New Client"** from the dashboard
2. Fill in Client Name (required), CMMS Type, CRx Consultant, Client POC, and Overall Status
3. Click **"Create Client"** — the client appears on the dashboard with all tasks defaulted to *Not Started*

### Managing Integration Status
1. Click any client card to open the detail view
2. Under the **Integration Status** tab, use the dropdown on each task row to update its status
3. Changes are saved instantly — the dashboard stats update in real time

### Adding Comments to a Task
1. In the Integration Status tab, click the **💬 Comments** button on any task row
2. Enter your name and comment, then click **"Post Comment"**
3. Comments are timestamped and stored permanently per task

### Adding Client Notes
1. Open a client → click the **Client Notes** tab
2. Enter your name and note, click **"Post Note"**
3. Notes appear in chronological order with timestamps

### Adding Documents
1. Open a client → click the **Documents** tab
2. Click **"+ Add Document"** and fill in type, name, URL link, and any notes
3. Document links are clickable and open in a new tab

### Exporting to PDF
1. Open any client detail page
2. Click **"↓ PDF"** in the top right
3. A formatted PDF downloads with all tasks, statuses, and documents

---

## 🗂️ Project Structure

```
CRx-integration-tracker/
│
├── index.html          ← The entire application (rename from integration-tracker.html)
└── README.md           ← This file
```

That's it. No `node_modules`, no build step, no config files.

---

## 💾 Data Storage

All data is stored in your **browser's localStorage** under the key `CRx_tracker_v2`.

- Data is **per-browser** — opening the app in Chrome vs Firefox = separate data
- Data is **per-device** — your work laptop and home laptop won't sync automatically
- Data is **not affected by updates** to the HTML file
- To back up your data: open browser DevTools → Application → Local Storage → copy the value of `CRx_tracker_v2`

---

## 🔧 Customization

Want to add more task items to the integration template? Find the `defaultSections()` function in the `<script>` block and add entries to the `tasks` arrays. New tasks will appear for all newly created clients (existing clients are unaffected).

---

## 🛠️ Tech Stack

- **Vanilla HTML/CSS/JavaScript** — zero frameworks, zero dependencies
- **localStorage API** — for persistent data storage
- **jsPDF** (CDN) — for PDF generation
- **Google Fonts** — Syne + DM Mono typefaces

---

## 📄 License

Internal tool — for CRx team use.
