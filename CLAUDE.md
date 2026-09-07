# CLAUDE.md

## What this repo is

Public, static privacy-policy/support pages for Saliko Solutions' apps, hosted free via GitHub Pages at:
https://webbdylan.github.io/turf-measurement-optimizer-policies/

It exists so App Store (and similar) submissions have a Privacy Policy URL / Support URL without exposing any app's actual source code — this repo is intentionally public and contains only these pages, nothing else.

No dependencies. No build step. Plain HTML/CSS/vanilla JS, deployed as-is by GitHub Pages from the `main` branch root. Don't introduce a bundler, framework, or package.json for this — it doesn't need one.

## Structure

- `style.css` — shared styling for every page in this repo (light/dark mode aware). All apps reuse this one file.
- `index.html` — root directory page listing every app hosted here, linking to each app's privacy policy + support page.
- `privacy-policy.html`, `support.html` (repo root) — **Turf Measurement Optimizer**'s pages. These live at the root (not in a subfolder) because that's where they were first created, and the URLs are already referenced live in App Store Connect for that app's submission.
  - **Do not rename, move, or delete these two files or change their paths.** Doing so breaks the Privacy Policy / Support URLs Apple has on file for that app. Content edits are fine; path changes are not.
- `/<app-name>/privacy-policy.html`, `/<app-name>/support.html` — every *other* app gets its own subfolder. This is the pattern for all apps added after Turf Measurement Optimizer.

## Adding a new app

1. Create `/<app-name>/` with `privacy-policy.html` and `support.html`, following the same structure/tone as the Turf Measurement Optimizer pages (see root `privacy-policy.html` for the template: what data section, no-account section, contact section, etc.) — tailor the "what the app uses and why" section to what that specific app actually does. Don't copy-paste data usage claims from another app; get them right for this one.
2. Reference the shared stylesheet with a relative path: `<link rel="stylesheet" href="../style.css">`.
3. Support page's contact form reuses the **same shared Apps Script endpoint** (URL is in the `<script>` block of the root `support.html` — copy it verbatim) and the **same shared Google Sheet**. Give the form a hidden field identifying the app:
   ```html
   <input type="hidden" name="app" value="Your App Name Here">
   ```
   The Apps Script's `doPost` already logs a `params.app` column, so submissions from every app land in one sheet, distinguishable by that column.
4. Add a `<div class="card">` entry to root `index.html` linking to the new app's pages (see the commented-out example block already in that file).
5. Commit, push. GitHub Pages redeploys automatically (usually live within ~30–60 seconds).

## Contact form / backend

There is no server other than a single Google Apps Script Web App deployment (bound to a Google Sheet), shared across all apps in this repo. It:
- Accepts `POST` with form fields: `contact`, `reason`, `details`, `app`, and a honeypot `website` field (must stay empty — non-empty means spam, request is silently ignored).
- Has no `doGet` logic beyond a static placeholder — actual submissions must be `POST`, not `GET`.
- The deployed Web App URL only lives in each `support.html`'s inline `<script>` — there's no shared config file for it. If it's ever redeployed as a **new** deployment (not a new version of the same deployment), the URL changes and every `support.html` across every app needs updating.

Quirk worth knowing: `curl`/non-browser requests to the Apps Script URL often return a misleading Google-generated error page (e.g. "Page Not Found") even when the request actually succeeds server-side. Don't trust curl's response body to judge success/failure — check the linked Google Sheet directly, or test from an actual browser. The deployed form itself uses `fetch(..., { mode: "no-cors" })`, so it never depends on reading that response anyway.
