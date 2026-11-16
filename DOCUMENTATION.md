# EP Central — Time Tracking (Clockify Replacement): Dev Spec

Updated after the meeting with Daim (Nov 2026). This replaces the earlier two-phase plan that included a Clockify import wizard and a dashboard. Both are now out of scope.

## 1. Decisions from the meeting

| # | Decision | What it means for the build |
|---|----------|-----------------------------|
| 1 | No Clockify import | Daim backs up Clockify before the switch. The team starts fresh in EP Central. No import wizard, entity mapping or Clockify API client. |
| 2 | No dashboard | Only Timesheet, Reports and Projects. |
| 3a | Time on a task | Inside a task (Activity → Time), start a timer or add time manually, choosing a Project (required) and a Company (optional). |
| 3b | Time without a task | A new **Timesheet** tab with a timer or manual entry. **Project is required; company is optional.** Time with no company goes to the default company **EP (Internal)**. |
| 4 | Export | PDF (EP branded) and Excel only. No CSV. |
| 5 | No timestamps in reports | Reports and exports show date and duration only, with no start or end times. Entries still record start and end. |
| 6 | Approval before reports | A report can't be exported until every entry in it is approved. |
| 7 | Batch onboarding | Current TMS users move over first, then the rest of the team in batches. |

## 2. Domain model

- **Company**: the portfolio client (Cambium, Moby Filters…). A seeded default company, `EP (Internal)`, holds internal work.
- **Project**: a service workstream (B&A Retainer Services, Monthly Close…). Each Project belongs to one **Product** (Accounting or FP&A; internal projects use `Internal`) and is assigned to **many** Companies (many-to-many).
- **Time entry**: one person, one Project (required), an optional Company (defaults to EP (Internal)), an optional Task, a date, start and end times, a duration, a description and an approval status.

### Backend changes

`backend/tasks/models/task_relations.py`, `TimeLog`:
- `task`: make nullable (`null=True, blank=True`) so that Timesheet entries have no task.
- `project = ForeignKey('tasks.Project', on_delete=PROTECT, related_name='time_logs')`: required on every entry.
- `company = ForeignKey('company.Company', null=True, on_delete=SET_NULL, related_name='time_logs')`. If no company is given, set the default `EP (Internal)` company on save.
- `duration_seconds = PositiveIntegerField(default=0)`, to store `HH:MM:SS` precision.
- Approval fields:
  - `status = CharField(choices=pending/approved/rejected, default='pending', db_index=True)`
  - `approved_by = ForeignKey(User, null=True)`, `approved_at = DateTimeField(null=True)`
  - `rejected_by`, `rejection_reason = TextField(blank=True)`
- `updated_at = DateTimeField(auto_now=True)`

`backend/tasks/models/project.py`, `Project`:
- Add `companies = ManyToManyField('company.Company', related_name='assigned_projects', blank=True)`.
- Note: `Project` has no `clockify_id` field today. Since there's no import, none is needed.

Seed data: the `EP (Internal)` company and the internal projects `General / Internal` and `Training & Team Meetings`.

Removed from the earlier plan: `clockify_entry_id`, `external_user_email`/`external_user_name`, `ClockifyImportJob` and the Clockify API client. These only existed for the import.

## 3. Approval rules

- Any user who is **Lead or above** and sits **above the person in the reporting line** (direct or indirect manager) can approve or reject that person's entries.
- Nobody approves their own time. A Lead's time is approved by the person above them.
- Time logged by the top of the hierarchy, who has no manager, is **auto-approved** *(assumption: confirm with Daim)*.
- **Reject** needs a reason. The owner sees the reason, edits the entry, and the entry goes back to `pending`.
- **Editing an approved entry** sends it back to `pending`; the UI warns before saving. Approved entries show a lock icon.
- **Report gate**: the export is locked if any entry in the filtered report is `pending` or `rejected`. The banner shows:
  - the number of unapproved entries and their total hours;
  - which approvers are holding things up;
  - a **Bulk approve** button for anything the viewer can approve.
- **Bulk approve** groups the pending entries the viewer can approve by **person + company + project**. The lead ticks groups and approves them in one go. It's available from Reports (scoped to the current filters) and from Timesheet → Team approvals (scoped to the selected week).

The hierarchy needs a `manager` (reports-to) link on the user or profile, plus a role rank (Analyst < Lead < Manager). Check whether TMS already stores this. If it doesn't, it's a new field.

## 4. Screens (see the prototype)

### Timesheet (`Time Tracking → Timesheet`)
- Clockify-style entry bar with "What are you working on?", a project picker, and a Timer or Manual mode toggle. Manual mode takes a date and start and end times; an end time before the start time counts as overnight.
- Entries are listed by day with a status pill, start–end times, duration, and edit and delete buttons for the owner. Approved entries can't be deleted.
- **Team approvals** (Lead and above only): pending, rejected and approved entries from people below you, with **Approve** and **Reject** on each entry, plus **Approve all pending**.

### Task time (`Task → Activity → Time`)
- The same grouped project picker, defaulting to the task's company.
- A timer, or manual start and end times.
- The entries table shows who logged each entry and its status. Owners can edit; approvers can approve.

### Grouped project picker (used everywhere)
It mirrors Clockify's project filter:
- **Project headers** (the workstreams), with **one Company row under each Project it's assigned to**.
- Searching a company name lists it once under every project it belongs to; searching a project name lists all of that project's companies.
- Terminology follows the Clockify mapping: **Clockify Client → EP Project**, **Clockify Project → EP Company**.
- Single-select in the entry forms. **Project is required**: Start, Add, Log and Save are blocked until one is picked. **Company is optional**: clicking the project header ("· no company") saves the entry against **EP (Internal)**. Multi-select with checkboxes in the report filter.

### Reports (`Time Tracking → Reports`)
- **Filters**: date range, user, company, project (grouped picker), product, description keyword.
- **Summary**: total, a daily bar chart, a breakdown grouped by Project, Company, User or Product (with a sub-grouping), and a donut chart.
- **Detailed**: description, Company · Project, task key, user, **date**, approval status, duration. **No start or end times.**
- **Weekly**: Company · Project (or User) × day matrix with totals. A month range shows one column per week.
- **EXPORT** button with *Save as PDF (EP branded)* and *Save as Excel*, both for the active tab and locked until everything is approved.

### Projects (`Time Tracking → Projects`)
The directory of projects, their product, assigned companies, logged time and the share of time approved.

## 5. Export formats

The column layouts follow Clockify's standard exports, with start and end times removed:

| Report | Excel sheets and columns | PDF |
|--------|--------------------------|-----|
| Summary | **Summary**: Project, Company, Product, Entries, Time (h), Time (decimal), Total row. **By day**: Date, Time (h), Time (decimal) | Branded header, daily bar chart, grouped table with duration, decimal hours and share |
| Detailed | **Detailed**: Date, Company, Project, Product, Task, Description, User, Duration (h), Duration (decimal), Approved by | Branded table: Date, Company · Project, Description, Team member, Duration |
| Weekly | **Weekly**: Company, Project (or User), one column per day, Total (h), Total (decimal), Total row | Branded matrix |

Every file starts with a header block: report type, date range, filters, and total approved time. File names follow `EP_Time_Report_<Type>_<MM_DD_YYYY-MM_DD_YYYY>.xlsx`.

## 6. API endpoints

- `GET /api/tasks/projects/?company_id=<id>`: projects with their assigned companies, for the picker.
- `GET/POST /api/tasks/time-logs/`: Timesheet entries (filters: `user`, `from`, `to`, `status`; `task` optional).
- `POST /api/tasks/<task_id>/time-logs/`, `PATCH /api/tasks/time-logs/<id>/`: editing an approved entry resets it to pending.
- `POST /api/tasks/time-logs/<id>/approve/`, `POST /api/tasks/time-logs/<id>/reject/` (`{reason}`)
- `POST /api/tasks/time-logs/bulk-approve/`: `{ids: [...]}` or `{user_id, company_id, project_id, from, to}`. The server only approves entries the caller is allowed to approve.
- `GET /api/tasks/time-reports/{summary|detailed|weekly}/`
- `GET /api/tasks/time-reports/export/?type=summary|detailed|weekly&format=pdf|xlsx`: returns **409** with the list of unapproved entries if anything in scope isn't approved.

All permission checks (who can approve, the export gate) must run on the server; the UI only mirrors them.

## 7. Edge cases still relevant

1. **Overnight entries**: store timezone-aware `started_at`/`ended_at`; daily and weekly grouping uses the workspace timezone (`TruncDate(..., tzinfo=...)`).
2. **Running timers**: an entry with no end time is still in progress. Leave it out of reports and approvals until it's stopped.
3. **Hierarchy changes**: when someone's manager changes, their pending entries move to the new approver automatically, because approval rights are checked at action time, not stored per entry.
4. **Deleted or deactivated users**: their approved time stays in reports; their pending entries need an admin path to approve or reject.

## 8. Open questions

1. Is the top of the hierarchy auto-approved, or does someone else (an admin) approve that time?
2. Should approval also be possible by **week per person**, or is the entry and person · company · project grouping enough?
3. Can approved entries be edited at all, or should they be fully locked so that only an approver can unlock them?
