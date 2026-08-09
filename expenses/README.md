# Expense tracker

`index.html` is a standalone, self-contained personal expense tracker. Like
`profile/`, it has no build dependency on the rest of this repository (a fork
of [github/docs](https://github.com/github/docs)) — open the file directly in
a browser, or serve `expenses/` as a static site (e.g. via GitHub Pages).

## What it does

- Log expenses with date, amount, currency, category, payment method,
  description, and an optional receipt photo.
- Filter and search the expense list by date range, category, payment
  method, or description text.
- See a running total, this month's total, and a per-category spend
  breakdown.
- Edit or delete any entry.
- Export all data to CSV or JSON, or import a previously exported JSON file.

## Data storage

All data is stored in the browser's `localStorage`, under the key
`expenseTracker:v1` — nothing is sent to a server. This means:

- Data is local to one browser on one device; it does not sync across
  devices or browsers.
- Clearing site data/local storage for this page deletes the data.
- Receipt photos are stored inline as base64 data URLs (capped at 1 MB per
  image) as part of each entry, so large receipt libraries can approach
  `localStorage`'s per-origin size limit (typically 5–10 MB). Export to JSON
  periodically as a backup, especially if attaching many receipts.
