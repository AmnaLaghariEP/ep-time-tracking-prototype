# Enduring Planet TMS — Time Tracking, Analytics & Reports Prototype

An interactive, high-fidelity frontend prototype built for the **Accounting & FP&A Team** to demonstrate multi-project time tracking, editing logged time, executive analytics dashboards, weekly matrix reports, and Enduring Planet branded exports.

---

## 🚀 Live Demo & Screens Included

1. **Task Time Tracking (Add Time & Choose Project)**:
   - Modeled after the Enduring Planet Central Task Drawer for *"Monthly Accounting Task Tracker - October 2026"*.
   - Demonstrates that a single broad tracker task can track time across multiple client projects (*Cambium*, *Dollaride*, *Portmanteau*, *Carbon Crew*, *Emrgy*, *Turtle Hill*, *Enduring Planet*).
   - **Live Ticking Timer**: Start Timer with ticking clock, stop and auto-log.
   - **Manual Entry**: Start/End time, date, user, duration, and notes.

2. **View & Edit Logged Time on Tasks**:
   - Complete itemized list of time entries logged on the task with client project tags.
   - **Interactive Edit Modal**: Click the pencil icon on any entry to re-assign the project, modify start/end times or durations, edit notes, or delete.
   - All edits immediately synchronize across the entire application and reports in real time!

3. **Executive Dashboard**:
   - Modeled after the Clockify Executive Overview.
   - Top KPI Cards: Total Hours (`695:14:50`), Top Project (`Enduring Planet - Accounting`), Top Client (`B&A Retainer Services`).
   - Stacked Daily Activity Bar Chart (Mon–Sun).
   - Donut Chart & Ranked Project Breakdown with progress bars and percentage tags.

4. **Summary Report**:
   - Faithfully recreated from Clockify summary screenshots.
   - Date range selector, client & project filter badges, total hours (`26:40:00`).
   - Daily activity bar chart (Mon `06:30`, Tue `07:30`, Wed `04:20`, Thu `02:20`, Sun `06:00`).
   - Twin Donut visualizers: By Project (100% Cambium) & By Description breakdown.
   - Grouped duration summary table.

5. **Detailed Report**:
   - Line-by-line timesheet matching Clockify Detailed Report.
   - Project colored tags, billable `$` indicators, user avatars, timestamp ranges, and durations.
   - Inline edit triggers.

6. **Weekly Report Matrix**:
   - Matrix table showing Project row with columns for `Mo, Oct 12`, `Tu, Oct 13`, `We, Oct 14`, `Th, Oct 15`, `Fr, Oct 16`, `Sa, Oct 17`, `Su, Oct 18`, and `Total`.
   - Summed totals per day and grand total.

7. **Enduring Planet Branded PDF / Print Export**:
   - Replaces Clockify logo with official **Enduring Planet SVG Logo**.
   - Clean, professional client billing report layout with print styling (`window.print()`).

---

## 📦 How to Run Locally

You can open `index.html` directly in any web browser:

```bash
# On macOS:
open index.html

# Or start a simple local server if preferred:
npx serve .
# or
python3 -m http.server 8080
```

---

## 🌐 How to Publish to GitHub Pages (Like Taban's Prototype)

To publish this prototype to your GitHub account (e.g. `https://<your-username>.github.io/ep-time-tracking-prototype/`):

1. **Create a new repository on GitHub**:
   - Name: `ep-time-tracking-prototype` (or any name you prefer).
   - Make it Public.

2. **Push this folder to GitHub**:
   ```bash
   cd ep-time-tracking-prototype
   git init
   git add .
   git commit -m "Initial commit: Enduring Planet Time Tracking prototype"
   git branch -M main
   git remote add origin https://github.com/<your-username>/ep-time-tracking-prototype.git
   git push -u origin main
   ```

3. **Enable GitHub Pages**:
   - Go to your repository on GitHub.
   - Click **Settings** ➔ **Pages** (under Code and automation in the left sidebar).
   - Under **Build and deployment** ➔ **Source**: Choose **Deploy from a branch**.
   - Under **Branch**: Select `main` and folder `/ (root)`, then click **Save**.
   - Within 60 seconds, your site is live at:
     `https://<your-username>.github.io/ep-time-tracking-prototype/`
