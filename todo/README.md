# To-do list

`index.html` is a standalone, self-contained personal to-do list. Like
`profile/` and `expenses/`, it has no build dependency on the rest of this
repository (a fork of [github/docs](https://github.com/github/docs)) — open
the file directly in a browser, or serve `todo/` as a static site (e.g. via
GitHub Pages).

## What it does

- Add tasks with a title, optional due date, priority (low/medium/high),
  category, and notes.
- Mark tasks done/active with one click; overdue and due-today tasks are
  flagged.
- Switch between Active, Completed, and All views.
- Filter by category or priority, search by title/notes, and sort by due
  date, priority, or date added.
- See active/completed/overdue/total counts and a completion progress bar.
- Edit or delete any task; clear all completed tasks at once.
- Export all data to JSON, or import a previously exported JSON file.

## Data storage

All data is stored in the browser's `localStorage`, under the key
`todoList:v1` — nothing is sent to a server. This means:

- Data is local to one browser on one device; it does not sync across
  devices or browsers.
- Clearing site data/local storage for this page deletes the data.
- Export to JSON periodically as a backup, especially before clearing
  browser data.
