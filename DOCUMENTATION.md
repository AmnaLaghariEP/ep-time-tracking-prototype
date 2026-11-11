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

## 3. Backend Edge Cases, Loopholes & Engineering Safeguards (Dev Checklist)

An audit of the live Clockify workspace data and the existing Django importer (`backend/tasks/services/clockify_import.py`) identified **7 critical edge cases** that the backend implementation must handle:

### Edge Case 1: Mixed / Inverted `Client` vs. `Project` Naming Inside Clockify
- **The Problem**: In most of the Clockify workspace, `Project` is the Portfolio Company (e.g., `Alchemy CO2`, `Emrgy`, `Erg Bio`) and `Client` is the Service Workstream (e.g., `B&A Retainer Services`, `AP Management`, `Cash Budget`). However, some historical entries have the Company under `Client` (e.g., `Cambium`, `Cambium Carbon PBC`) and the workstream under `Project` (`Understanding system/Correcting EP mistake`), or combine both with a colon (`Portmanteau: Underwriting Framework`).
- **Backend Solution**:
  - In `resolve_entities()` (Step 1 -> Step 2), match both Clockify `client_name` and `project_name` against `Company.objects` and `Project.objects` (case-insensitive + alias table).
  - In `ClockifyImportJob.field_mappings` (Step 2), allow per-row/per-pair role assignment (`company_source: 'project' | 'client'`) so the Admin can flip any inverted historical entry before execution.

### Edge Case 2: Duplicate / Variant Company Names (`Cambium` vs. `Cambium Carbon PBC`)
- **The Problem**: `clockify_import.py` (line 185) currently uses `Company.objects.get(name__iexact=name)` and auto-creates a new `Company` when an exact string match fails. Because Clockify contains variants like `Cambium` and `Cambium Carbon PBC` (or `Moby Filter` vs. `Moby Filters Inc`), blind auto-creation will create duplicate companies in `company_company`.
- **Backend Solution**:
  - Never auto-create a `Company` during `execute_import()` unless the Admin explicitly selects `"Create New Company"` in Step 2 (`field_mappings['companies'][clockify_name] = {'action': 'map', 'company_id': existing_id}`).
  - Support many-to-one alias mapping so multiple Clockify strings (`"Cambium"`, `"Cambium Carbon PBC"`) resolve to the same `company_id`.

### Edge Case 3: Entries with `(Without client)` / `NO CLIENT` (`335:52:05` hrs) & Internal EP Work (`235:17:48` hrs)
- **The Problem**: Over 168 entries (`335:52:05` hours) in Clockify have `client = null` (`(Without client)`), and `235:17:48` hours are logged under `Enduring Planet - Accounting` (internal company work).
- **Backend Solution**:
  - Allow `TimeLog.project` to fall back to a default `Uncategorized / General` `Project` (under `Accounting` or `FP&A`) when Clockify's workstream is null.
  - Seed or map an `Enduring Planet (Internal)` `Company` record so internal accounting/FP&A hours are preserved and reportable without polluting external client metrics.

### Edge Case 4: Batch Onboarding — Preserving Hours of Users Not Yet in TMS
- **The Problem**: `clockify_import.py` (lines 298–301) currently skips any row where the Clockify user's email is not found in `users_user` (`if not user_info['found']: rows_skipped += 1; continue`). Because users are onboarded to TMS in batches, importing all hours for a Company will drop hours logged by analysts who haven't been onboarded yet or former contractors—causing EP Central's company totals to be lower than Clockify's.
- **Backend Solution**:
  - Make `TimeLog.user` nullable (`null=True, blank=True`) and add `external_user_email = models.EmailField(blank=True, default='', db_index=True)` and `external_user_name = models.CharField(max_length=150, blank=True, default='')` on `TimeLog`.
  - Add a `post_save` signal on `User` creation that backfills `TimeLog.objects.filter(user__isnull=True, external_user_email__iexact=instance.email).update(user=instance)`.

### Edge Case 5: Active / Running Timers at the Moment of Import
- **The Problem**: When fetching entries via the Clockify REST API (`GET /v1/workspaces/{workspaceId}/user/{userId}/time-entries`), any timer currently running has `timeInterval.end = null` and `timeInterval.duration = null`, which causes duration parsing to fail.
- **Backend Solution**:
  - Filter out in-progress entries (`if not entry.get('timeInterval', {}).get('end'): continue`) during `fetch_clockify_entries()`.

### Edge Case 6: Idempotent Re-Imports & Updating Corrected Clockify Entries
- **The Problem**: During transition, an accountant may fix a typo in Clockify (e.g., changing `02:00:00` to `03:30:00`) and the Admin may re-run the import for that date range.
- **Backend Solution**:
  - Store `clockify_entry_id = models.CharField(max_length=64, unique=True, null=True, blank=True, db_index=True)` on `TimeLog`.
  - Use `TimeLog.objects.update_or_create(clockify_entry_id=entry_id, defaults={...})` during `execute_import()` so re-imports are 100% idempotent and automatically update durations/notes if modified in Clockify.

### Edge Case 7: Timezone & Overnight (`+1`) Shift Boundaries (PKT `UTC+5` vs. US Timezones)
- **The Problem**: Many shifts are overnight (`9:00PM - 3:00AM +1`). If `started_at` is stored or queried in naive UTC instead of the workspace timezone, an entry logged at `9:00 PM PKT` (`4:00 PM UTC`) vs. `3:00 AM PKT` (`10:00 PM UTC previous day`) can land on a different date bucket in the Summary and Weekly charts.
- **Backend Solution**:
  - Store timezone-aware `started_at` and `ended_at` (`DateTimeField`) on `TimeLog`, and group daily/weekly report aggregations using `TruncDate('started_at', tzinfo= ZoneInfo(settings.CLOCKIFY_WORKSPACE_TIMEZONE))`.

---

## 4. Task-Mapping Implementation Options (Pending Product Sign-Off)

In `backend/tasks/services/clockify_import.py` (lines 308–313), `TimeLog` currently requires a `task_id`. Once the Accounting team confirms their preferred option in the Demo Guide, implement one of the following in `execute_import()`:

1. **Option 1 (Attach to Monthly Tracker Task)**: Resolve `(company, month, product)` -> attach `TimeLog` to the company's active monthly recurring tracker `Task` (e.g., `Cambium - Task Tracker - October 2026`), setting `TimeLog.project = resolved_project` and `TimeLog.description = clockify_description`.
2. **Option 2 (Task per Description)**: `Task.objects.get_or_create(company=company, project=project, title__iexact=clockify_description)` and attach `TimeLog` to that task.
3. **Option 3 (Nullable `TimeLog.task`)**: Set `TimeLog.task = None`, `TimeLog.company = company`, `TimeLog.project = project` for historical Clockify imports, while requiring `task` only for time logged directly inside the EP Central Task Drawer.



