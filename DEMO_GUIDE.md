# ⏱️ EP Central — Clockify & Time Tracking Integration Demo Guide
**Walkthrough & Requirement Alignment for Accounting & FP&A Team**

- **Live Interactive Demo**: https://amnalaghariep.github.io/ep-time-tracking-prototype/
- **Goal**: Bring Clockify's time tracking, reporting, and client exports directly into **EP Central (TMS)** while migrating all historical Clockify data without losing a single hour.

---

## 📅 Proposed Two-Phase Rollout

- **Phase 1 — Clockify Historical Import & Task-Level Time Tracking**:
  - Import existing Clockify data in batches (by Date Range, User, Company, or Project/Workstream).
  - Log and edit time directly inside EP Central tasks (`Activity ➔ Time` tab) with a **Project selector** and **Live Timer**.
- **Phase 2 — Clockify Dashboard, Reports & Branded Exports**:
  - Full Clockify-style **Dashboard**, **Summary Report**, **Detailed Report**, **Weekly Report**, and **Projects Directory** inside EP Central, with one-click **PDF / CSV / Excel exports**.

---

## 🖥️ Part 1: Live Prototype Walkthrough (What We Are Demoing)

### 1. Multi-Step Clockify Import (`Settings ➔ Clockify Integration`)
- **No API Key Needed in UI**: The Clockify connection is configured securely on the server.
- **Step 1 — Select Workspace, Date Range & Batch Scope**:
  - Choose **Time Range**: `This Month`, `This Week`, `All Time`, or `Custom Date Range`.
  - Choose **Onboarding Batch Scope**: Because we are onboarding team members and clients in batches, we can import:
    - *All Tasks & Time Entries*
    - *Only for a Specific Company*
    - *Only for a Specific Team Member (User)*
    - *Only for a Specific Project / Workstream*
- **Step 2 — Map Fields (Separate Screen)**:
  - Review and confirm how Clockify **Users**, **Companies**, and **Projects / Workstreams** (`Accounting` vs. `FP&A`) map into EP Central.
- **Step 3 — Preview & Confirm (Separate Screen)**:
  - Filterable preview table with checkboxes to review exact entries and total hours before importing.

---

### 2. Logging & Editing Time Inside EP Central Tasks (`Tasks ➔ Task Board ➔ EPA-1881`)
- In EP Central, a single recurring monthly task (e.g., `EPA-1881: Moby Filter - Task Tracker - October 2026`) often covers work across multiple different projects/workstreams.
- Inside the Task Modal under **`Activity ➔ Time`**:
  - **Choose Project**: Select which Project/Workstream you are working on (e.g., *Monthly Close*, *Understanding system/Correcting EP mistake*, *Ad-hoc Financial Advisory*).
  - **Live Timer or Manual Entry**: Start/stop the live timer or enter Start/End times and a description note.
  - **Edit Any Entry**: Click **Edit** on any logged time row to update its Project, Date, Duration, or Note.

---

### 3. Clockify Dashboard, Reports & Exports (`Clockify` Sidebar Section)
- **Dashboard (`Clockify ➔ Dashboard`)**:
  - Matches Clockify's Executive Dashboard: **Total Time**, **Top Project**, **Top Client**, **7-Day Stacked Bar Chart**, **Donut Chart**, and **Project % Breakdown**.
- **Reports (`Clockify ➔ Reports`)**:
  - **Multi-Filter Bar**: Filter by Date Range, Team Member, Company, Project, Product (`Accounting` / `FP&A`), and Description keyword.
  - **Summary Report**: 7-day vertical green bar chart + expandable project breakdown (`[count]` badge + total duration) + multi-color donut chart.
  - **Detailed Report**: Line-by-line time entry table (`Description`, `Company - Project`, `Billable $`, `User`, `Time & Date (+1 overnight indicator)`, `Duration`).
  - **Weekly Report**: 7-day (`Mon`–`Sun`) matrix by project and user.
  - **Export Dropdown (`EXPORT ▾`)**: Download filtered reports as **EP-Branded Client PDF**, **CSV**, or **Excel (`.xlsx`)**.
- **Projects Directory (`Clockify ➔ Projects`)**:
  - View all Projects, their linked Product (`Accounting` or `FP&A`), and assigned Companies (supporting many-to-many relationships where one Company has multiple Projects, and standard Projects are shared across Companies).

---

## ❓ Part 2: Questions for Team Alignment (To Finalize Before Development)

### Question 1: Clockify `Client` vs. `Project` — Standardizing Company & Project in TMS
- **What we noticed in Clockify**:
  - In most of Clockify, **Clockify `Project` is the Portfolio Company** (e.g., `Alchemy CO2`, `Compost Colorado`, `Emrgy`, `Erg Bio`) and **Clockify `Client` is the Service Workstream** (e.g., `B&A Retainer Services`, `FP&A Retainer Services`, `AP Management`, `Cash Budget`).
  - **However, some entries are reversed**: in the `Client` dropdown, some actual Companies (like `Cambium` and `Cambium Carbon PBC`) appear as Clockify `Clients`, while their workstream (`Understanding system/Correcting EP mistake`) is entered as the Clockify `Project`. Other entries combine both in one name (e.g., `Portmanteau: Underwriting Framework`).
- **Questions for Daim's Team**:
  1. In EP Central (TMS), we want to standardize so **Company = Portfolio Company** (e.g., `Cambium`, `Alchemy CO2`) and **Project = Service Workstream** (e.g., `B&A Retainer Services`, `AP Management`, `Cash Budget`) linked to a **Product** (`Accounting` or `FP&A`). Does this structure work for the team?
  2. For the mixed entries in Clockify (like `Cambium` under `Client`), would you prefer to clean those up inside Clockify before we import, **or** should our Step 2 (`Map Fields`) screen let the Admin flip/assign whether an imported item is a **Company** or a **Project**?

---

### Question 2: How Should Imported Clockify Entries Connect to TMS Tasks? (Choose 1 of 3 Options)
In Clockify, time entries only have `Client`, `Project`, `Description` (e.g., `Categorization of Bank statement of 4401`), and `User / Duration` — they don't have a separate Clockify "Task" field filled in. **When we import these entries into EP Central, which of the 3 options below do you prefer?**

- **Option 1 — Attach to the Company's Monthly Task Tracker (Recommended for Recurring Accounting)**:
  - **How it works**: In EP Central, each company has a monthly recurring task (e.g., `Cambium - Task Tracker - October 2026`). All imported Clockify entries for `Cambium` in October 2026 attach as **Time Logs** inside that monthly task (under `Activity ➔ Time`), tagged with their `Project` and `Description`.
  - **Benefit**: Keeps the Kanban Task Board clean (no clutter from hundreds of daily descriptions), while showing all hours both inside the Company's monthly Task Modal **and** in Clockify Dashboard/Reports.
- **Option 2 — Create a Separate TMS Task Card on the Board for Every Description**:
  - **How it works**: Each Clockify **Time Entry Description** (e.g., `Categorization of Bank statement of 4401`) creates or matches a separate Task card on the TMS Task Board.
  - **Trade-off**: If accountants type slightly different descriptions every day in Clockify, this will create hundreds of small task cards on the Kanban board.
- **Option 3 — Keep Historical Clockify Logs in Reports/Dashboard Only (No Task Card Needed)**:
  - **How it works**: Historical imported Clockify entries are saved directly under `(Company + Project + User)` so they appear in **Clockify Dashboard & Reports**, without creating or modifying cards on the TMS Task Board. Going forward, when the team logs time inside EP Central tasks, those new entries attach to the task.

---

### Question 3: How Should We Handle `(Without client)` (`335+ hrs`) & Internal EP Hours (`235+ hrs`)?
- **What we noticed in Clockify**:
  - In the Summary Report, **168 entries (`335:52:05` hours)** are logged under **`(Without client)` / `NO CLIENT`** (e.g., `Fieldston Dev pledgor 2`).
  - On the Dashboard, the largest item is **`Enduring Planet - Accounting` (`235:17:48` hours)** for internal EP work.
- **Questions for Daim's Team**:
  1. When an entry in Clockify has **`(Without client)`**, should we assign it to a default project (e.g., `General / Uncategorized`) so those 335+ hours are included in reports?
  2. For internal hours (`Enduring Planet - Accounting`), should we have an **`Enduring Planet (Internal)`** company option in TMS?

---

### Question 4: Merging Duplicate Company Names (`Cambium` vs. `Cambium Carbon PBC`)
- **What we noticed in Clockify**:
  - Clockify has both `Cambium` and `Cambium Carbon PBC` (and similarly `Moby Filter` vs. `Moby Filters Inc`).
- **Question for Daim's Team**:
  - During import (`Step 2: Map Fields`), should we merge both `Cambium` and `Cambium Carbon PBC` into the single official **`Cambium Carbon PBC`** company in EP Central?

---

### Question 5: Batch Onboarding — Historical Hours from Analysts Not Yet in TMS
- **Question for Daim's Team**:
  - As we onboard team members in batches, if we import all historical hours for a company (like `Cambium`), some entries may belong to an analyst who hasn't been onboarded to TMS yet (or a past team member). Should we **import 100% of those hours** under their name/email so the company's total hours in EP Central match Clockify 100% (and auto-link to their profile once they join TMS)?

---

### Question 6: Transition Plan — Cutover vs. Parallel Logging
- **Question for Daim's Team**:
  - Once a batch of users/clients is imported into EP Central, will that group switch **100% to logging time inside EP Central**, or will they continue logging in Clockify for a transition period (requiring periodic re-imports)?
