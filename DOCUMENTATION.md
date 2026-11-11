# Enduring Planet (EP Central) — Time Tracking & Clockify Integration
**Team Presentation & Technical Architecture Plan (Phase 1 & Phase 2)**

---

## Executive Overview

As Enduring Planet transitions the Accounting and FP&A teams from standalone Clockify usage into **EP Central (TMS)**, our goal is to provide a seamless, zero-data-loss migration and a native time-tracking experience inside EP Central.

- **Interactive Prototype Demo**: `index.html`
- **Rollout Strategy**: Divided into two clear engineering phases:
  - **Phase 1 — Clockify API Import, Entity Mapping & Task-Level Multi-Project Time Tracking (Backend + Core Task UI)**
  - **Phase 2 — Clockify Executive Dashboard, Reports (Summary, Detailed, Weekly) & Branded Exports (PDF / CSV / Excel)**

---

## Phase 1: Clockify Import, Entity Mapping & Task Time Tracking

### 1.1 Business & Workflow Requirements (Phase 1)

1. **Multi-Project Time Tracking Inside EP Central Tasks**:
   - In EP Central, tasks are often general recurring trackers (e.g., `EPA-1881: Moby Filter - Task Tracker - October 2026`).
   - A team member working on that task may log hours across **multiple different projects** (e.g., *Monthly Close*, *Understanding system/Correcting EP mistake*, *Ad-hoc Financial Advisory*).
   - Inside the **EP Central Task Modal (`Activity > Time` tab)**, users can:
     - Select the **Project** from a dropdown before starting the **Live Timer** or logging **Start / End** times manually.
     - View all logged time entries on the task and click **Edit** to update the Project, Date, Duration, or Note at any time.

2. **Domain Relationships & Inverted Clockify Naming Convention (`Company` <-> `Project` -> `Product`)**:
   - **Important Clockify Workspace Discovery**: In our team's Clockify workspace, the `Client` and `Project` fields are used in an inverted hierarchy:
     - **Clockify `Project` = EP Central `Company`** (e.g., *Alchemy CO2*, *Compost Colorado*, *Emrgy*, *Erg Bio*, *Cambium*, *Dollaride*, *Bedrock Energy*, *Moby Filters*).
     - **Clockify `Client` = EP Central `Project` (Service Workstream)** (e.g., *B&A Retainer Services*, *FP&A Retainer Services*, *AP Management*, *Cash Budget*, *Fund Model FP&A*, *Investment Operations*, *Accounting Audit*, *Understanding system/Correcting EP mistake*).
   - **EP Project -> EP Product (Many-to-One)**: Every EP Project (imported from a Clockify `Client`) belongs to a specific EP Product line (`Accounting` or `FP&A`).
   - **EP Company <-> EP Project (Many-to-Many)**: A single EP Company (imported from a Clockify `Project`) can have multiple EP Projects, and standard EP Projects (such as *B&A Retainer Services*, *FP&A Retainer Services*, *AP Management*, or *Cash Budget*) are shared across multiple EP Companies.
   - **Auto-Match / Auto-Create on Import**: During Clockify import, Clockify `Project` names automatically match to existing EP Companies (case-insensitive) or create a new EP Company if unmatched, while Clockify `Client` names map to EP Projects linked to `Accounting` or `FP&A`.

3. **Jira-Style 4-Screen Clockify Import Wizard (`Settings > Clockify Integration`)**:
   - **No UI API Key Input**: The Clockify API key is stored securely in backend environment variables (`settings.CLOCKIFY_API_KEY`), never exposed in the UI.
   - **Step 1 — Select Workspace & Batch Scope**:
     - Select Clockify Workspace (`Enduring Planet — Accounting & FP&A`).
     - Select **Time Range**: `This Month`, `This Week`, `All Time`, or `Custom Date Range`.
     - Select **Onboarding Batch Scope**: Because users, companies, and projects are onboarded to TMS in batches, admins can choose:
       - *Import All Tasks & Time Entries*
       - *Import for Certain Company (Clockify Project)*
       - *Import for Certain User (Team Member)*
       - *Import for Certain Project / Workstream (Clockify Client)*
   - **Step 2 — Map Fields (Separate Screen)**:
     - Review & override auto-matched **Users** (by email), **Clockify Projects -> EP Companies**, and **Clockify Clients -> EP Projects & Products (`Accounting` / `FP&A`)**.
   - **Step 3 — Preview & Confirm (Separate Screen)**:
     - Filterable preview table with row-level checkboxes to include/exclude specific time entries before committing.
   - **Step 4 — Import Complete**:
     - Summary of imported entries, matched companies, mapped projects, and audit log confirmation.

---

### 1.2 Current Backend Gaps & Required Backend Changes (Phase 1)

An audit of the current Django backend (`backend/tasks/`) highlights four required backend changes for Phase 1:

#### 1. Upgrade `TimeLog` Model (`backend/tasks/models/task_relations.py`)
- **Current State**: `TimeLog` only has `task`, `user`, `duration_minutes`, `description`, `logged_at`, `started_at`, and `ended_at`. It has no `project` field, assuming each `Task` only has one project.
- **Required Changes**:
  - Add `project = models.ForeignKey('tasks.Project', on_delete=models.SET_NULL, null=True, blank=True, related_name='time_logs')` so each time log entry records which specific Project was worked on.
  - Add `company = models.ForeignKey('company.Company', on_delete=models.SET_NULL, null=True, blank=True, related_name='time_logs')` and allow `task` to be nullable (or auto-linked to the company's active monthly tracker task).
  - Add `duration_seconds = models.PositiveIntegerField(default=0)` to preserve exact Clockify `HH:MM:SS` precision.
  - Add `clockify_entry_id = models.CharField(max_length=64, unique=True, null=True, blank=True, db_index=True)` for idempotent deduplication across repeated batch imports.
  - Add `is_billable = models.BooleanField(default=True)` and `updated_at = models.DateTimeField(auto_now=True)`.

#### 2. Add Many-to-Many `companies` on `Project` (`backend/tasks/models/project.py`)
- **Current State**: `Project` already has `product = ForeignKey('tasks.Product')`, but only has a single `company = ForeignKey('company.Company')`.
- **Required Changes**:
  - Add `companies = models.ManyToManyField('company.Company', related_name='assigned_projects', blank=True)` on `Project` so standard projects (linked to `Accounting` or `FP&A` Products) can be assigned across multiple Companies, and each Company can have multiple Projects.

#### 3. Invert `Client`/`Project` Mapping & Fix Task Skipping in `backend/tasks/services/clockify_import.py`
- **Current State**:
  - `clockify_import.py` currently maps Clockify `Client` -> EP `Company` and Clockify `Project` -> EP `Project`. However, in our actual Clockify workspace, **Clockify `Project` is the Company** (e.g., *Alchemy CO2*, *Compost Colorado*, *Emrgy*, *Erg Bio*, *Cambium*) and **Clockify `Client` is the Project / Service Workstream** (e.g., *B&A Retainer Services*, *FP&A Retainer Services*, *AP Management*, *Cash Budget*).
  - Additionally, in `clockify_import.py` (lines 308–313), if a Clockify entry does not have an explicit `task` column value, the import skips the row (`no task resolved`).
- **Required Changes**:
  - Invert the entity resolver in `clockify_import.py` so **Clockify `Project` -> EP `Company`** and **Clockify `Client` -> EP `Project` (linked to `Product`: `Accounting` or `FP&A`)**.
  - If no explicit task name is present on the Clockify entry, either match the company's active monthly tracker task for that month/product or attach the `TimeLog` directly to `(company, project)` so zero valid Clockify entries are skipped.

#### 4. Direct Clockify REST API Client & Multi-Step `ClockifyImportJob`
- **Current State**: `ClockifyImportJob` (`backend/tasks/models/clockify_import.py`) only handles static CSV uploads.
- **Required Changes**:
  - Add `backend/tasks/clockify/client.py` authenticating via `settings.CLOCKIFY_API_KEY` from `.env`.
  - Update `ClockifyImportJob` states to `FETCHING -> MAPPING -> READY -> PROCESSING -> COMPLETED` (matching `JiraImportJob`), storing `workspace_id`, `time_range`, `scope_mode` (`all`, `client`, `user`, `project`), and `field_mappings`.

---

### 1.3 Phase 1 API Endpoints Summary

| Method | Endpoint | Purpose |
| :--- | :--- | :--- |
| `GET` | `/api/tasks/clockify/workspaces/` | Fetch Clockify workspaces, clients, users, and projects using server `CLOCKIFY_API_KEY` |
| `POST` | `/api/tasks/clockify/import/fetch/` | **Step 1 -> Step 2**: Fetch Clockify entries for selected Time Range & Batch Scope and return entity resolution map |
| `POST` | `/api/tasks/clockify/import/<job_id>/map/` | **Step 2 -> Step 3**: Save confirmed User, Company, and Project -> Product mappings; return preview rows |
| `POST` | `/api/tasks/clockify/import/<job_id>/execute/` | **Step 3 -> Step 4**: Bulk-import selected `TimeLog` entries idempotently (`clockify_entry_id`) |
| `POST` | `/api/tasks/<task_id>/time-logs/` | Log time (manual or live timer stop) on a Task with a selected `project_id` |
| `PATCH` | `/api/tasks/<task_id>/time-logs/<log_id>/` | **Edit** an existing logged time entry (`project_id`, `started_at`, `ended_at`, `duration_seconds`, `description`) |
| `GET` | `/api/tasks/projects/?company_id=<id>` | List Projects available for a given Company (including cross-company M2M Projects) |

---

## Phase 2: Executive Dashboard, Reports & Branded Exports

Once Phase 1 is live and all historical + new time entries are stored in `TimeLog` with accurate `Company`, `Project`, and `Product` (`Accounting` vs. `FP&A`) links, Phase 2 delivers the reporting and export suite inside EP Central.

### 2.1 Business & Workflow Requirements (Phase 2)

1. **Clockify Section in EP Central Sidebar**:
   - **Dashboard**: Executive KPIs (Total Hours, Accounting Hours, FP&A Hours, Active Clients).
   - **Reports**: Summary, Detailed, Weekly, and Enduring Planet Branded PDF Preview.
   - **Projects**: Directory of Projects, linked Products, and assigned Companies.

2. **Multi-Filter Bar Across All Reports**:
   - Filter simultaneously by **Date Range**, **User**, **Client (EP Company)**, **Project**, **Product** (`Accounting`, `FP&A`), and **Keyword**.

3. **Three Core Report Views + Branded Export Engine**:
   - **Summary Report**: Daily bar chart, project donut chart, and grouped `Company — Project` breakdown.
   - **Detailed Report**: Line-by-line audit log (`Description`, `Client • Project`, `Billable ($)`, `User`, `Date & Time Range`, `Duration`).
   - **Weekly Report**: 7-day matrix (`Mon` – `Sun` + `Total`) grouped by Project/User.
   - **Export Dropdown (`EXPORT ▾`)**: Exports strictly respect active filters (`Save as PDF (EP Branded)`, `Save as CSV`, `Save as Excel`).

### 2.2 Backend Aggregation & Export Endpoints (Phase 2)

| Method | Endpoint | Purpose |
| :--- | :--- | :--- |
| `GET` | `/api/tasks/time-reports/summary/` | Returns total hours, `Accounting` vs. `FP&A` split, daily bar-chart buckets, and grouped `(company, project, product)` totals |
| `GET` | `/api/tasks/time-reports/detailed/` | Paginated list of `TimeLog` records filtered by `start_date`, `end_date`, `user_id`, `company_id`, `project_id`, `product_id`, and `search` |
| `GET` | `/api/tasks/time-reports/weekly/` | Returns 7-day (`Mon`–`Sun`) aggregated matrix per Project/User for the selected week |
| `GET` | `/api/tasks/time-reports/export/` | Generates downloadable **CSV**, **Excel (`.xlsx`)**, or **EP-Branded PDF** using the active filter parameters |

---

## 3. Open Decision Question for the Team: How Should Imported Clockify Entries Map to TMS Tasks?

### Context
In Clockify, most time entries only contain **4 fields** (and usually **no** separate Clockify "Task" field), and in our workspace **Clockify `Project` represents the Company** while **Clockify `Client` represents the Project / Service Workstream**:
1. **Clockify `Project` (Company)** (e.g., `Alchemy CO2`, `Emrgy`, `Cambium`) -> maps to **EP Company** (`Alchemy CO2`, `Emrgy`, `Cambium LLC`)
2. **Clockify `Client` (Project / Service Workstream)** (e.g., `B&A Retainer Services`, `AP Management`, `Cash Budget`, `FP&A Retainer Services`, `Understanding system/Correcting EP mistake`) -> maps to **EP Project** (linked to **Product: `Accounting` or `FP&A`**)
3. **Time Entry Description** (e.g., `Categorization of Bank statement of 4401`)
4. **User & Duration** (e.g., `Hassan`, `02:00:00`)

In the current backend code (`backend/tasks/services/clockify_import.py`, lines 308–313), `TimeLog` requires a `task_id`. Because Clockify entries do not have a separate `task` column filled in, the existing importer skips them (`"no task resolved"`).

### Question for Team Alignment
**When we import time entries from Clockify into EP Central (TMS), how should those entries be linked to TMS Tasks? Please choose from the 3 options below:**

#### Option 1 — Map to the Company's Monthly Task Tracker (Recommended for Recurring Accounting)
- **How it works**: In EP Central, each company already has a monthly recurring task (e.g., `Cambium - Task Tracker - October 2026`). When importing Clockify entries for `Cambium` in October 2026, all imported entries attach as **Time Logs** inside `Cambium - Task Tracker - October 2026` (under `Activity -> Time`), tagged with their respective `Project` (`Understanding system/Correcting EP mistake`) and `Description` (`Categorization of Bank statement of 4401`).
- **Pros**: Keeps the TMS Kanban Task Board clean (no clutter from hundreds of one-off descriptions), while ensuring all hours appear both inside the Company's monthly Task Modal **and** in Clockify Dashboard/Reports.

#### Option 2 — Create or Match a TMS Task from the Time Entry Description
- **How it works**: Use the Clockify **Time Entry Description** (e.g., `Categorization of Bank statement of 4401`) as the TMS Task title. If a task with that title already exists under `Cambium`, attach the time log to it; otherwise, create a new TMS Task on the board and attach the time log.
- **Pros / Cons**: Every distinct description becomes a task card on the board, which provides granular task cards but can flood the Kanban board with hundreds of small tasks if accountants type free-form daily descriptions in Clockify.

#### Option 3 — Store Historical Clockify Logs Directly Against `(Company + Project + User)` (No Task Required)
- **How it works**: Make `TimeLog.task` optional (`null=True`) and store `company` + `project` directly on `TimeLog`. Historical Clockify imports go straight into the `TimeLog` table for **Clockify Dashboard & Reports** without creating or modifying cards on the TMS Task Board. Going forward, when users log time directly inside EP Central tasks, those new entries will have `task_id` populated.
- **Pros**: Complete separation between historical Clockify reporting data and active TMS Kanban task cards.

