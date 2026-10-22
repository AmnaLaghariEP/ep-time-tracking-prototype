# Enduring Planet Central — Tasks & Clockify Time Tracking Prototype

A high-fidelity standalone frontend prototype replicating the exact **Enduring Planet Central (EP Central)** user interface, prepared for the Accounting team demo (Josh, Daim, etc.).

---

## 🖥️ Live Screens & Features Included

1. **Exact EP Central Shell & Navigation**:
   - **Top Brand Bar**: Emerald green header with "EP Central" branding, global search, `99+` notification badge, and `AL` user avatar.
   - **Left Sidebar**: Matches EP Central navigation:
     - `Tasks` ➔ `Tasks Board` (Kanban board) & `Product Roadmap`
     - `Companies`, `Users`, `Reports`, `Messages`, `Management`, `Inbox (631)`
     - **`Clockify`** (New Dedicated Section):
       - `Dashboard`
       - `Reports` (Summary, Detailed, Weekly Matrix, EP Branded PDF)
       - `Projects`
     - **`Settings`**:
       - `Clockify Integration` (4-step API connection & migration wizard)
     - `Definitions`

2. **EP Central Tasks Board (Kanban View)**:
   - Matches the live EP Central board layout:
     - Columns: `Open Task`, `Monthly (2)`, `Weekly (0)`, `Under Review - Team Lead`, `Under Review - Client`, `Done`.
     - Cards: `EPA-1881 (Moby Filter - Task Tracker - October 2026)` and `EPA-1845 (September 2026)`.
     - Clicking **`EPA-1881`** opens the real Task Drawer modal!

3. **Task Modal (`EPA-1881`) with Exact `Activity > Time` Tab**:
   - Matches the screenshots provided by the team:
     - Title, `Monthly` status badge, description, and checklist of 6 subtasks.
     - Right details sidebar: `Assigned To: Daim Ashraf / AL`, `Company: Moby Filters Inc`, `Product: Accounting`, `Priority: High`, `Start/Due Dates`, `Estimated: 40h`, `Total Logged: 18h 45m`.
     - **Activity Navigation**: Tabs for `Time`, `Comments (4)`, `Email`, `Attachments`, `History`.
     - **Log Time Form**:
       - **Multi-Project Support**: Project selector dropdown allows tracking time against any client project (`Moby Filters - Monthly Close`, `Ad-hoc Advisory`, `Payroll Audit`, etc.) or adding a new project on the fly.
       - **Manual Entry**: Start date/time (`06:52 PM`), end date/time, note input, and "Log Time" button.
       - **Live Timer**: Interactive stopwatch mode (`▶ Start Timer` / `⏹ Stop & Log`).
     - **Editable Logged Time Table**:
       - Itemized list of all past logs.
       - **[Edit]** button: Allows editing project, duration, date, or note with instant updates to total hours.
       - **[Delete]** button: Removes an entry.

4. **Clockify Executive Dashboard**:
   - Modeled after Clockify's workspace analytics:
     - KPI Cards: Workspace Hours (`695:14:50`), Billable Utilization (`96.8%`), Active Clients (`12`), Active Projects (`28`).
     - Stacked daily activity bar chart (Mon–Sun).
     - Client distribution donut chart (`Cambium 21.3%`, `Moby Filters 16.1%`, `Dollaride 13.9%`, etc.).

5. **Clockify Reports**:
   - Sub-tabs for:
     - **Summary Report**: Client summary table with active projects and hours.
     - **Detailed Timesheet**: Line-by-line filterable log with verified records from Daim Ashraf.
     - **Weekly Matrix**: Project row with daily columns (Mon–Sun) and row totals.
     - **EP Branded PDF Preview**: Official invoice/timesheet with Enduring Planet logo and print stylesheet (`window.print()`).

6. **Settings > Clockify Integration (API Import Flow)**:
   - 4-step wizard demonstrating how Clockify data will be ingested:
     - **Step 1**: API Key Authentication & Workspace selection (`Daim Ashraf's Workspace`).
     - **Step 2**: **Client $\rightarrow$ Company Mapping**:
       - Enforces the strict rule: *Clockify clients map strictly to existing EP Central Companies (no auto-creation of companies to prevent database pollution).*
     - **Step 3**: **Project $\rightarrow$ Product Mapping**:
       - Explains auto-matching existing projects or auto-creating projects under `Accounting` or `FP&A`.
     - **Step 4**: One-click historical import trigger with live progress indicator.

---

## 🏃 How to Run Locally

You can open `index.html` directly in any web browser:

```bash
# On macOS:
open /Users/amna/EP/ep-time-tracking-prototype/index.html

# Or run a local HTTP server:
cd /Users/amna/EP/ep-time-tracking-prototype
python3 -m http.server 8080
# then open http://localhost:8080
```

---

## 🌐 How to Publish to GitHub Pages (Live URL)

To share this demo with Josh and Daim via a public link (e.g. `https://<your-username>.github.io/ep-time-tracking-prototype/`):

1. **Create a new repository on your GitHub**:
   - Name: `ep-time-tracking-prototype`
   - Set to **Public**

2. **Push this folder to GitHub**:
   ```bash
   cd /Users/amna/EP/ep-time-tracking-prototype
   git remote add origin https://github.com/<your-username>/ep-time-tracking-prototype.git
   git push -u origin main
   ```

3. **Enable GitHub Pages**:
   - Open your repo on GitHub ➔ **Settings** ➔ **Pages**.
   - Under **Build and deployment** ➔ **Branch**: Choose `main` and `/ (root)`, then click **Save**.
   - Your live prototype will be ready within 1 minute!
