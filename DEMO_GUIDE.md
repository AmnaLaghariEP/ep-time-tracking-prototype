# EP Central Time Tracking — Team Demo Guide

Live prototype: https://amnalaghariep.github.io/ep-time-tracking-prototype/

All names, companies and hours in the prototype are made-up demo data.

## What changed after the meeting with Daim

- **Removed**: the Clockify import wizard and the dashboard. Daim backs up Clockify, and the team starts fresh in EP Central.
- **Added**: a **Timesheet** tab for time without a task, an **approval flow**, Clockify-style **grouped project search**, and **PDF and Excel** exports for the Summary, Detailed and Weekly reports.
- **Reports no longer show start or end times**, only date and duration.

## Before you start: "Viewing as"

The top-right **Viewing as** dropdown switches who you are, so you can show what each role sees:

| Person | Role | Reports to |
|--------|------|------------|
| Daim Ashraf | Manager | — (auto-approved) |
| Ahmed Raza | Lead | Daim |
| Farzeen Ahmed | Lead | Daim |
| Hassan, Bilal Qureshi | Analyst | Ahmed |
| Sana Malik | Analyst | Farzeen |

The demo opens as **Ahmed Raza (Lead)**.

## Walkthrough

### 1. Log time without a task (Timesheet)
1. Go to **Time Tracking → Timesheet**.
2. Type a description and open the **Project** picker. Search for a company, e.g. `dolla`: Dollaride appears once under each project it's assigned to (B&A Retainer Services, AP Management, Cash Budget…).
3. Click **Start**, then **Stop**. The entry shows up as **Pending**.
4. Switch to Manual (the list icon), set a start and end time, and click **Add**.
5. Try adding an entry with no project: it's blocked, because **project is required**. Then click a project header (e.g. **B&A Retainer Services · no company**) and add it. The entry goes to **EP (Internal)**, because company is optional.

### 2. Log time on a task
1. Click **Open Task EPA-1881**.
2. In **Activity → Time**, pick a Project (and optionally a company), then start the timer or enter start and end times and click **Log**.
3. The table shows who logged each entry, its status, and Edit or Approve.

### 3. Approve time (Lead and above)
1. In Timesheet, click **Team approvals**. You see pending entries from Hassan and Bilal (Ahmed's team).
2. Use **Approve** or **Reject** on one entry. Rejecting asks for a reason, which the analyst then sees.
3. Or click **Approve all pending** to bulk-approve by person + company + project.
4. Switch **Viewing as → Hassan**. There's no Team approvals tab; Hassan sees only his own time and its status.

### 4. Reports are locked until time is approved
1. Go to **Time Tracking → Reports** (Last week).
2. The amber banner says the export is locked and shows how many entries aren't approved. **EXPORT** shows a lock.
3. As Ahmed, click **Bulk approve**, keep the groups ticked, and click **Approve selected**. The banner turns green.
4. Viewing as Hassan, the banner instead says the entries are *waiting on Ahmed Raza*, with no approve button.

### 5. Export PDF and Excel
1. With everything approved, open **Summary**, **Detailed** or **Weekly**.
2. **EXPORT → Save as PDF (EP branded)** opens a branded preview; **Save as PDF** prints it.
3. **EXPORT → Save as Excel** downloads a real `.xlsx` (e.g. `EP_Time_Report_Weekly_10_12_2026-10_18_2026.xlsx`).
4. Neither format has start or end times.

### 6. Projects
**Time Tracking → Projects** lists each project with its product (Accounting, FP&A or Internal), the companies assigned to it, its logged time and the share of time approved.

## Questions to confirm with the team

1. Is the Manager at the top of the hierarchy auto-approved, or does an admin approve that time?
2. Is approving per entry, plus bulk approval by person · company · project, enough, or do leads also want to approve a whole week per person?
3. When an approved entry is edited, is "goes back to Pending" right, or should approved entries be fully locked?
