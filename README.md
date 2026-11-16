# Enduring Planet Central — Time Tracking Prototype

A standalone, front-end-only prototype of time tracking inside EP Central, which replaces Clockify for the Accounting and FP&A teams. All data in it is made-up demo data.

See [DEMO_GUIDE.md](DEMO_GUIDE.md) for the team walkthrough and [DOCUMENTATION.md](DOCUMENTATION.md) for the dev spec.

## What's in it

- **Timesheet** (`Time Tracking → Timesheet`): log time without a task, with a timer or manual start and end times. **Project is required, company is optional**; time with no company goes to **EP (Internal)**.
- **Team approvals**: Leads and above approve or reject entries from anyone below them in the reporting line, one at a time or in bulk.
- **Task time** (`Open Task EPA-1881 → Activity → Time`): timer or manual entry on a task, with a Project · Company picker.
- **Grouped project picker**: Clockify-style. Project headers with one row per assigned company; searching a company lists it under every project it belongs to.
- **Reports**: Summary, Detailed and Weekly, with no start or end timestamps. Export is **locked until every entry in the report is approved**, with a bulk-approve shortcut for leads.
- **Exports**: an EP-branded PDF (print preview) and a real Excel `.xlsx` for each report type.
- **Projects**: projects, their product, assigned companies (many-to-many), logged time and approval share.
- **Viewing as** switcher (top right) to demo the Analyst, Lead and Manager views.

Removed after the meeting with Daim: the Clockify import wizard, the dashboard, and CSV export.

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
