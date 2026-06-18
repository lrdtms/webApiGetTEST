# Design: GitHub Pages API Tester

**Date:** 2026-06-18  
**Status:** Approved

## Overview

A single `index.html` page hosted on GitHub Pages that lets a user test both Dashboard API endpoints interactively. No build step, no dependencies, no server. Plain HTML, CSS, and JavaScript.

## Target API

Defined in `dashboard-api.md`. Two GET endpoints:

- `GET /api/dashboard/summary?date=YYYY-MM-DD`
- `GET /api/dashboard/clockevents?from=YYYY-MM-DD&to=YYYY-MM-DD`

All requests require `X-Api-Key: <DASH key>` header.

## Pre-requisite

The GitHub Pages origin (`https://lrdtms.github.io`) must be added to the API's CORS allowlist before browser-based calls will succeed.

## Layout

Three stacked sections in one `index.html`:

### 1. Config Bar (top)
- **Base URL** text input — e.g. `https://myapi.azurewebsites.net`
- **API Key** password input — masked by default
- Shared by both endpoint sections; no submit button here

### 2. Summary Card
- **Date** date picker (maps to `date` query param)
- **Get Summary** button
- Results table columns: Cohort | Total Students | Clocked In | Not Clocked In
- Each row is clickable to expand/collapse two sub-lists: clocked-in students and not-clocked-in students (name + surname)
- Inline status/error area below the button

### 3. Clock Events Card
- **From** and **To** date pickers
- **Get Clock Events** button
- Results table columns: Student | Cohort | Date | Clock In | Clock Out
- Null clock times render as `—`
- Inline status/error area below the button

## Data Flow

1. User fills Base URL and API Key in the config bar
2. User sets date(s) for an endpoint and clicks the fetch button
3. JS validates inputs client-side (both fields required); blocks request if empty
4. `fetch()` call constructed: `${baseUrl}${path}${queryString}` with `X-Api-Key` header
5. Response JSON parsed and rendered into the relevant table
6. No data is stored or persisted anywhere — every click re-fetches live

## Error Handling

| Scenario | Display |
|---|---|
| Empty base URL or API key | Inline: "Base URL and API Key are required" — request not sent |
| HTTP 401 / 403 | Inline: API's `message` field from the error JSON body |
| Other non-2xx | Inline: `HTTP {status}: {statusText}` |
| Network failure / CORS block | Inline: "Could not reach the API — check the base URL and that your origin is on the CORS allowlist" |
| Empty array response | Inline: "No data returned for this date / range" — no table rendered |

## Deployment

1. Create a new GitHub repo (e.g. `dashboard-api-tester`)
2. Push `index.html` to `main`
3. Enable GitHub Pages in repo settings → source: `main` branch, `/ (root)`
4. Site available at `https://lrdtms.github.io/dashboard-api-tester/`
5. Add that origin to the API CORS allowlist

## Out of Scope

- Authentication / login (API key entered manually each session)
- Data persistence or caching
- POST/PUT/DELETE endpoints
- Mobile-specific layout optimisation
