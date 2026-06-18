# GitHub Pages API Tester Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single `index.html` page on GitHub Pages that lets a user enter a base URL and API key, then call and display results from both Dashboard API endpoints.

**Architecture:** One self-contained HTML file with inline CSS and JavaScript — no build step, no dependencies, no server. All state lives in DOM inputs; nothing is persisted. Two card sections share one `apiFetch` helper that reads the config bar on every call.

**Tech Stack:** Plain HTML5, CSS, vanilla JavaScript (`fetch` API). Deployed via GitHub Pages.

## Global Constraints

- No external JS libraries or CSS frameworks — everything inline in `index.html`
- No data persistence — every button click re-fetches live
- API key field must use `type="password"` (masked)
- Base URL trailing slash is stripped before constructing request URLs
- Network errors (`TypeError`) must show the CORS allowlist message; all other errors show the HTTP status + API message
- Null `clockInTime` / `clockOutTime` values render as `—`

---

### Task 1: Scaffold `index.html` with config bar and CSS

**Files:**
- Create: `index.html`

**Interfaces:**
- Produces: `document.getElementById('baseUrl').value` — base URL string used by all fetch calls
- Produces: `document.getElementById('apiKey').value` — API key string used by all fetch calls

- [ ] **Step 1: Create `index.html`** with the following complete content:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Dashboard API Tester</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body { font-family: system-ui, sans-serif; background: #f5f5f5; color: #222; padding: 24px; }
    h1 { font-size: 1.4rem; margin-bottom: 20px; }
    h2 { font-size: 1rem; margin-bottom: 12px; color: #444; }

    .config-bar {
      background: #fff;
      border: 1px solid #ddd;
      border-radius: 8px;
      padding: 16px;
      display: flex;
      gap: 16px;
      flex-wrap: wrap;
      margin-bottom: 24px;
    }
    .config-bar label {
      display: flex;
      flex-direction: column;
      gap: 4px;
      font-size: 0.85rem;
      font-weight: 600;
      flex: 1;
      min-width: 200px;
    }
    .config-bar input {
      padding: 8px;
      border: 1px solid #ccc;
      border-radius: 4px;
      font-size: 0.9rem;
    }

    .card {
      background: #fff;
      border: 1px solid #ddd;
      border-radius: 8px;
      padding: 20px;
      margin-bottom: 24px;
    }
    .form-row {
      display: flex;
      gap: 12px;
      flex-wrap: wrap;
      align-items: flex-end;
      margin-bottom: 12px;
    }
    .form-row label {
      display: flex;
      flex-direction: column;
      gap: 4px;
      font-size: 0.85rem;
      font-weight: 600;
    }
    .form-row input[type="date"] {
      padding: 8px;
      border: 1px solid #ccc;
      border-radius: 4px;
      font-size: 0.9rem;
    }

    button {
      padding: 8px 18px;
      background: #2563eb;
      color: #fff;
      border: none;
      border-radius: 4px;
      font-size: 0.9rem;
      cursor: pointer;
    }
    button:hover { background: #1d4ed8; }

    .status {
      font-size: 0.85rem;
      color: #c0392b;
      min-height: 20px;
      margin-bottom: 12px;
    }

    table { width: 100%; border-collapse: collapse; font-size: 0.875rem; }
    th {
      text-align: left;
      padding: 8px 12px;
      background: #f0f4f8;
      border-bottom: 2px solid #ddd;
      font-weight: 600;
    }
    td { padding: 8px 12px; border-bottom: 1px solid #eee; }
    tr.expandable { cursor: pointer; }
    tr.expandable:hover td { background: #f9f9f9; }
    tr.detail td { background: #fafbfc; font-size: 0.82rem; color: #555; }
  </style>
</head>
<body>
  <h1>Dashboard API Tester</h1>

  <div class="config-bar">
    <label>
      Base URL
      <input type="text" id="baseUrl" placeholder="https://myapi.azurewebsites.net">
    </label>
    <label>
      API Key
      <input type="password" id="apiKey" placeholder="DASH...">
    </label>
  </div>

  <script>
  </script>
</body>
</html>
```

- [ ] **Step 2: Open `index.html` in a browser** (double-click the file or drag to browser)

  Expected: Page shows "Dashboard API Tester" heading, two input fields (Base URL + masked API Key), white card on grey background. No JS errors in the browser console.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: scaffold index.html with config bar and CSS"
```

---

### Task 2: Add shared fetch helper and Summary card

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `document.getElementById('baseUrl').value`, `document.getElementById('apiKey').value` from Task 1
- Produces: `apiFetch(path)` — async function, throws `Error` on validation/HTTP failure, throws `TypeError` on network failure, returns parsed JSON on success

- [ ] **Step 1: Add the Summary card HTML.** Inside `<body>`, after the `.config-bar` div and before `<script>`, insert:

```html
  <div class="card">
    <h2>GET /api/dashboard/summary</h2>
    <div class="form-row">
      <label>Date <input type="date" id="summaryDate"></label>
      <button onclick="fetchSummary()">Get Summary</button>
    </div>
    <div class="status" id="summaryStatus"></div>
    <div id="summaryTable"></div>
  </div>
```

- [ ] **Step 2: Add JS inside the `<script>` tag:**

```js
async function apiFetch(path) {
  const baseUrl = document.getElementById('baseUrl').value.trim().replace(/\/$/, '');
  const apiKey = document.getElementById('apiKey').value.trim();
  if (!baseUrl || !apiKey) throw new Error('Base URL and API Key are required');
  const res = await fetch(`${baseUrl}${path}`, {
    headers: { 'X-Api-Key': apiKey }
  });
  if (!res.ok) {
    let msg;
    try { const body = await res.json(); msg = body.message; } catch { msg = res.statusText; }
    throw new Error(`HTTP ${res.status}: ${msg || res.statusText}`);
  }
  return res.json();
}

async function fetchSummary() {
  const date = document.getElementById('summaryDate').value;
  const status = document.getElementById('summaryStatus');
  const container = document.getElementById('summaryTable');
  status.textContent = '';
  container.innerHTML = '';
  try {
    const data = await apiFetch(`/api/dashboard/summary?date=${date}`);
    if (!data.length) { status.textContent = 'No data returned for this date'; return; }
    renderSummaryTable(data, container);
  } catch (e) {
    status.textContent = e instanceof TypeError
      ? 'Could not reach the API — check the base URL and that your origin is on the CORS allowlist'
      : e.message;
  }
}

function renderSummaryTable(data, container) {
  const table = document.createElement('table');
  table.innerHTML = `<thead><tr>
    <th>Cohort</th><th>Total Students</th><th>Clocked In</th><th>Not Clocked In</th>
  </tr></thead><tbody></tbody>`;
  const tbody = table.querySelector('tbody');
  data.forEach(cohort => {
    const row = document.createElement('tr');
    row.className = 'expandable';
    row.innerHTML = `
      <td>${cohort.cohortName}</td>
      <td>${cohort.totalStudents}</td>
      <td>${cohort.totalClockedIn}</td>
      <td>${cohort.totalNotClockedIn}</td>
    `;
    const detail = document.createElement('tr');
    detail.className = 'detail';
    detail.hidden = true;
    const ci = cohort.clockedIn.map(s => `${s.name} ${s.surname}`).join(', ') || 'None';
    const nci = cohort.notClockedIn.map(s => `${s.name} ${s.surname}`).join(', ') || 'None';
    detail.innerHTML = `<td colspan="4">
      <strong>Clocked In:</strong> ${ci}<br>
      <strong>Not Clocked In:</strong> ${nci}
    </td>`;
    row.addEventListener('click', () => { detail.hidden = !detail.hidden; });
    tbody.appendChild(row);
    tbody.appendChild(detail);
  });
  container.appendChild(table);
}
```

- [ ] **Step 3: Verify — empty fields validation**

  Open `index.html` in browser. Leave Base URL and API Key empty. Click "Get Summary".
  Expected: Red status text reads "Base URL and API Key are required". No network request fires (check Network tab in DevTools).

- [ ] **Step 4: Verify — network/CORS error**

  Enter `https://doesnotexist.example.com` as Base URL, any text as API Key, pick any date. Click "Get Summary".
  Expected: Status text reads "Could not reach the API — check the base URL and that your origin is on the CORS allowlist".

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add apiFetch helper and Summary card"
```

---

### Task 3: Add Clock Events card

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `apiFetch(path)` from Task 2
- Consumes: `document.getElementById('baseUrl').value`, `document.getElementById('apiKey').value` from Task 1

- [ ] **Step 1: Add the Clock Events card HTML.** After the Summary card div, before `<script>`, insert:

```html
  <div class="card">
    <h2>GET /api/dashboard/clockevents</h2>
    <div class="form-row">
      <label>From <input type="date" id="clockFrom"></label>
      <label>To <input type="date" id="clockTo"></label>
      <button onclick="fetchClockEvents()">Get Clock Events</button>
    </div>
    <div class="status" id="clockStatus"></div>
    <div id="clockTable"></div>
  </div>
```

- [ ] **Step 2: Add JS inside the `<script>` tag, after the existing functions:**

```js
async function fetchClockEvents() {
  const from = document.getElementById('clockFrom').value;
  const to = document.getElementById('clockTo').value;
  const status = document.getElementById('clockStatus');
  const container = document.getElementById('clockTable');
  status.textContent = '';
  container.innerHTML = '';
  try {
    const data = await apiFetch(`/api/dashboard/clockevents?from=${from}&to=${to}`);
    if (!data.length) { status.textContent = 'No data returned for this date range'; return; }
    renderClockEventsTable(data, container);
  } catch (e) {
    status.textContent = e instanceof TypeError
      ? 'Could not reach the API — check the base URL and that your origin is on the CORS allowlist'
      : e.message;
  }
}

function renderClockEventsTable(data, container) {
  const table = document.createElement('table');
  table.innerHTML = `<thead><tr>
    <th>Student</th><th>Cohort</th><th>Date</th><th>Clock In</th><th>Clock Out</th>
  </tr></thead><tbody></tbody>`;
  const tbody = table.querySelector('tbody');
  data.forEach(ev => {
    const row = document.createElement('tr');
    row.innerHTML = `
      <td>${ev.name} ${ev.surname}</td>
      <td>${ev.cohortName}</td>
      <td>${ev.sessionDate}</td>
      <td>${ev.clockInTime ? formatTime(ev.clockInTime) : '—'}</td>
      <td>${ev.clockOutTime ? formatTime(ev.clockOutTime) : '—'}</td>
    `;
    tbody.appendChild(row);
  });
  container.appendChild(table);
}

function formatTime(isoString) {
  return new Date(isoString).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
}
```

- [ ] **Step 3: Verify — both cards visible**

  Open `index.html` in browser. Expected: Two cards visible — "GET /api/dashboard/summary" and "GET /api/dashboard/clockevents". No JS errors in console.

- [ ] **Step 4: Verify — Clock Events empty fields**

  Leave Base URL and API Key empty. Click "Get Clock Events".
  Expected: Status text reads "Base URL and API Key are required".

- [ ] **Step 5: Verify against live API (if API is accessible)**

  Enter real base URL and a valid DASH API key. Pick a valid date range for Clock Events. Click "Get Clock Events".
  Expected: Table renders with Student, Cohort, Date, Clock In, Clock Out columns. Any null clock times show `—` not `null`.

  Click a row in the Summary table. Expected: Row expands to show "Clocked In: ..." and "Not Clocked In: ..." lists. Click again to collapse.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add Clock Events card"
```

---

### Task 4: Deploy to GitHub Pages

**Files:**
- No code changes — deployment only

- [ ] **Step 1: Create a new public GitHub repo** named `dashboard-api-tester` at `https://github.com/new`. Leave it empty (no README).

- [ ] **Step 2: Push `index.html` to the new repo**

```bash
git remote add origin https://github.com/lrdtms/dashboard-api-tester.git
git push -u origin main
```

- [ ] **Step 3: Enable GitHub Pages**

  In the repo on GitHub: Settings → Pages → Source → Deploy from a branch → Branch: `main`, Folder: `/ (root)` → Save.

  GitHub will show a banner: "Your site is being published at `https://lrdtms.github.io/dashboard-api-tester/`". Wait ~60 seconds for the first deploy.

- [ ] **Step 4: Add the Pages origin to the API CORS allowlist**

  On the API side, add `https://lrdtms.github.io` as an allowed origin for the dashboard endpoints. The exact mechanism depends on your API configuration (e.g. `appsettings.json` CORS policy, middleware config). Without this step, all browser fetch calls will fail with a CORS error.

- [ ] **Step 5: Smoke test the live page**

  Open `https://lrdtms.github.io/dashboard-api-tester/` in a browser.
  - Enter your real base URL and a valid DASH API key.
  - Pick today's date for Summary → click Get Summary → table renders.
  - Pick a date range for Clock Events → click Get Clock Events → table renders.
  - Check browser DevTools → Network tab: requests show status 200, `X-Api-Key` header is present.
