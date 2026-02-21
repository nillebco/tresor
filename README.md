# trésor.

**Personal treasury forecast** — a zero-dependency, single-file web app for tracking and projecting your personal finances. No server, no account, no database. Everything lives in your browser's `localStorage`.

[Access this site](https://raw.githack.com/nillebco/tresor/main/index.html)

---

## Getting started

Open `index.html` in any modern browser. That's it.

---

## Entries

Entries are the core building block — each one represents an income or expense event.

### Adding an entry

Use the sidebar form to create an entry with the following fields:

| Field | Description |
|---|---|
| **Direction** | Income (▲) or Expense (▼), selected via toggle |
| **Label** | A short description, e.g. "salary", "rent" |
| **Amount** | Euro amount (positive number) |
| **Date** | Start date of the entry |
| **Frequency** | One-time, Monthly, or Yearly (see below) |
| **End Date** | Optional. Only available for recurring entries. |

### Frequencies

- **One-time** — appears exactly once on the given date.
- **Monthly** — repeats on the same day of the month (e.g. every 15th). Handles month-length edge cases (e.g. no Feb 30).
- **Yearly** — repeats on the same calendar date each year (e.g. every Jan 1).

### End dates for recurring entries

Recurring entries can optionally have an end date. Occurrences are only generated up to and including the end date. This is useful for modelling time-bounded costs or income — for example:

- AI subscription at €60/month until Dec 2025
- AI subscription at €80/month from Jan 2026 onward (no end date)

The end date is shown as a small hint (`→ MM/YYYY`) next to the label in the ledger.

### Editing an entry

Click any row in the ledger to load it into the sidebar form. The form switches to **Edit Entry** mode with a green "✓ Save Changes" button and a cancel option. The row being edited is highlighted in blue.

### Deleting an entry

Hover over any entry row — a ✕ button appears in the last column. Clicking it immediately deletes that entry. This works for both past and future entries.

---

## Ledger

The main panel shows a chronological ledger for the currently viewed year.

### Year navigation

Use the **‹** and **›** buttons in the top bar to move between years. The app opens on the current year by default.

### Row colour coding

Each row has a coloured left border indicating its type:

| Colour | Meaning |
|---|---|
| Blue border | Recurring income |
| Pink/red border | Recurring expense |
| Green border | One-time income |
| Amber border | One-time expense |

Future entries (dates after today) are displayed at reduced opacity to visually distinguish forecast from history.

### Month groupings

Entries are grouped by month with a sticky month header. At the end of each month, a **summary row** shows:

- Total income for that month
- Total expenses for that month
- Net (income − expenses), coloured green or red

### Running balance

The rightmost column shows the **running balance** after each transaction. This is a live projection that takes your latest observed balance as its starting point and applies all entries in chronological order.

---

## Observed Balances (Facts)

Facts let you anchor the forecast to reality by recording what your actual balance was on a specific date.

### Recording an observation

In the **Observed Balances** panel in the sidebar, enter a euro amount and a date (today or in the past), then click **+ Record Observation**. Multiple observations can be recorded over time — they accumulate as an audit trail.

### The anchor

The most recent observation whose date is ≤ today becomes the **anchor**. All balance calculations seed from the anchor:

- The running balance in the ledger starts from the anchor amount
- Only entries occurring *after* the anchor date are included in the forecast
- The forecasted balance card in the sidebar reflects anchor + all subsequent entries up to today

### Facts in the timeline

Observations appear as **green rows** directly in the ledger at their recorded date. When a fact row is encountered while reading the timeline, the running balance resets to the observed amount — because it's a real measurement, not a calculation.

- The active anchor is labelled `anchor` and shown in a brighter green
- Older facts are labelled `fact` and shown slightly dimmer

### Editing and deleting facts

- **Click** a fact row to edit its amount or date via a prompt. The date must be today or in the past.
- **Hover** to reveal the ✕ button and delete.

---

## Balance Card

The sidebar shows a **Forecasted Balance** card displaying your estimated current balance as of today. This is computed as:

```
anchor amount + Σ(all entry occurrences between anchor date and today)
```

If no observation has been recorded, the balance starts from zero.

---

## Stats Bar

The top of the ledger shows three summary figures for the viewed year:

| Stat | Description |
|---|---|
| **Total In** | Sum of all income entries in the viewed year |
| **Total Out** | Sum of all expense entries in the viewed year |
| **Year-end** | Projected balance at December 31 of the viewed year |

---

## Export & Import

### Export

Click **↓ Export JSON** to download a snapshot of all your data as a `.json` file named `treasury-YYYY-MM-DD.json`. The file contains:

```json
{
  "version": 2,
  "exportedAt": "<ISO timestamp>",
  "entries": [...],
  "facts": [...]
}
```

### Import

Click **↑ Import JSON** and select a previously exported file. If you already have data, you are prompted to choose between:

- **Merge** — new entries are added; existing entries with matching IDs are skipped
- **Replace** — all current data is discarded and replaced with the imported data

---

## Data Storage

All data is stored in the browser's `localStorage` under two keys:

| Key | Contents |
|---|---|
| `treasury_entries` | Array of entry objects |
| `treasury_facts` | Array of observed balance objects |

Data persists across sessions in the same browser. It is **not** synced across devices — use Export/Import to move data between browsers or machines.

The **Clear All Data** button in the sidebar permanently deletes all entries and observations after a confirmation prompt.

---

## Entry data model

```json
{
  "id": 1700000000000,
  "label": "salary",
  "amount": 3000,
  "date": "2025-01-15",
  "freq": "monthly",
  "direction": "in",
  "endDate": null
}
```

- `id` — Unix timestamp (milliseconds) used as a unique identifier
- `freq` — one of `"once"`, `"monthly"`, `"yearly"`
- `direction` — `"in"` (income) or `"out"` (expense)
- `endDate` — ISO date string or `null`

---

## Design notes

- Single `.html` file — no build step, no dependencies, no install
- Dark brutalist aesthetic with DM Mono and Syne fonts
- Currency formatted in French locale (`1 234,56`)
- Dates displayed in `DD/MM/YYYY` format
