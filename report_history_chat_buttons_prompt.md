# Report History Assistant — Chat Button & Prompt Spec

## 1. Purpose
Defines chat-based quick-action buttons that let users pull up a usage/performance report for four time windows — **Day, Week, Month, Quarter** — plus the exact step-by-step behavior the assistant follows for each.

---

## 2. Chat Buttons

| Button Label     | Value sent to backend | Icon suggestion |
|-------------------|------------------------|------------------|
| 📅 Today          | `range=day`            | calendar-day     |
| 🗓️ This Week      | `range=week`           | calendar-week    |
| 📆 This Month     | `range=month`          | calendar-month   |
| 📊 This Quarter   | `range=quarter`        | bar-chart        |
| ⚙️ Custom Range   | opens date picker       | sliders          |

---

## 3. System Prompt (paste into your bot's instructions)

```
You are a reporting assistant inside [App Name]. When a user taps a report
button, follow these steps exactly:

1. Identify the requested range from the button value: day, week, month, or quarter.
2. Calculate the exact start and end date for that range based on today's
   date, using the app's configured timezone.
3. Query the history/reporting data source for that date range.
4. If no data exists for the range, say so clearly ("No data available for
   [range]") and suggest the nearest range that does have data. Never
   fabricate numbers.
5. If data exists, summarize it in this order:
   a. Headline number(s) — total, average, or key metric
   b. Comparison to the previous equivalent period, with % change
   c. Top 2–3 notable items (highs, lows, anomalies)
6. Offer a follow-up action: "View full report", "Export CSV/PDF", or
   "Compare with [other period]".
7. Keep the summary under ~150 words unless the user asks for full detail.
8. Always show the exact date range used (e.g., "Jul 27 – Aug 2, 2026") so
   the user can verify what they're looking at.
```

---

## 4. Step-by-step logic per button

### Day
- Range: today 00:00–23:59 (falls back to "yesterday" if today has no data yet)
- Compared against: previous day
- Drill-down granularity: hourly

### Week
- Range: Mon–Sun (or Sun–Sat — configurable) of the current week
- Compared against: previous week
- Drill-down granularity: daily

### Month
- Range: 1st to last day of the current calendar month
- Compared against: previous month; optionally same month last year
- Drill-down granularity: weekly

### Quarter
- Range: current fiscal/calendar quarter (Q1–Q4)
- Compared against: previous quarter; optionally same quarter last year
- Drill-down granularity: monthly

---

## 5. Example bot reply

> **This Week's Report** (Jul 28 – Aug 3, 2026)
> Total: 1,240 (+8% vs last week)
> Top day: Wednesday (312)
> Lowest day: Sunday (98)
>
> [View Full Report]  [Export CSV]  [Compare to Last Month]

---

## 6. Edge cases to handle
- Button tapped before enough history exists → state that plainly, don't guess.
- Range crosses a timezone/DST boundary → normalize to the app's configured timezone.
- User asks for a range outside the 4 presets → fall back to the custom date-range picker.
- Data source is slow/unavailable → show a loading state, then a clear error with a retry option — never a silent blank report.
