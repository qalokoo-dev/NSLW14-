# NSLW — Report History Assistant: Chat Button & Prompt Spec

> Note: I've used **NSLW** as the app name based on your message. I wasn't sure what "W14" refers to (version tag, module/screen code, etc.) — let me know and I'll fold it in correctly.

## 1. Purpose
Defines chat-based quick-action buttons that let users pull up a usage/performance report for four time windows — **Day, Week, Month, Quarter** — with full step-by-step bot logic, and support for the **Ethiopian calendar** alongside Gregorian.

---

## 2. Chat Buttons

| Button Label     | Value sent to backend | Icon suggestion |
|-------------------|------------------------|------------------|
| 📅 Today          | `range=day`            | calendar-day     |
| 🗓️ This Week      | `range=week`           | calendar-week    |
| 📆 This Month     | `range=month`          | calendar-month   |
| 📊 This Quarter   | `range=quarter`        | bar-chart        |
| 🌐 Calendar Toggle | `calendar=ethiopian\|gregorian` | globe |
| ⚙️ Custom Range   | opens date picker       | sliders          |

---

## 3. System Prompt (paste into your bot's instructions)

```
You are the NSLW reporting assistant. When a user taps a report button,
follow these steps exactly:

1. Identify the requested range from the button value: day, week, month, or
   quarter.
2. Check the user's calendar preference (Ethiopian or Gregorian). Default to
   whichever the app profile has set; let the user switch via the Calendar
   Toggle button.
3. Calculate the exact start and end date for that range in the SELECTED
   calendar system, using the app's configured timezone.
4. Query the history/reporting data source for that date range (data is
   stored in Gregorian internally — convert only for display and range
   calculation).
5. If no data exists for the range, say so clearly ("No data available for
   [range]") and suggest the nearest range that does have data. Never
   fabricate numbers.
6. If data exists, summarize it in this order:
   a. Headline number(s) — total, average, or key metric
   b. Comparison to the previous equivalent period, with % change
   c. Top 2–3 notable items (highs, lows, anomalies)
7. Offer a follow-up action: "View full report", "Export CSV/PDF", or
   "Compare with [other period]".
8. Keep the summary under ~150 words unless the user asks for full detail.
9. Always show the exact date range used, in the active calendar — e.g.,
   "Meskerem 1–7, 2018 EC (Sep 11–17, 2025 GC)" — so the user can verify.
```

---

## 4. Step-by-step logic per button

### Day
- Range: today 00:00–23:59 (falls back to "yesterday" if today has no data yet)
- Compared against: previous day
- Drill-down granularity: hourly
- Ethiopian note: date label converts directly (day-to-day sequence is the same in both calendars, only the label changes)

### Week
- Range: Mon–Sun (or Sun–Sat — configurable) of the current week
- Compared against: previous week
- Drill-down granularity: daily
- Ethiopian note: weeks run identically to Gregorian (both use a continuous 7-day cycle) — only the displayed date labels differ

### Month
- **Gregorian mode:** 1st to last day of the current calendar month (28–31 days)
- **Ethiopian mode:** 1st to 30th of the current Ethiopian month — except **Pagumē**, the 13th month, which has 5 days (6 in an Ethiopian leap year)
- Compared against: previous month (in the same calendar system); optionally same month last year
- Drill-down granularity: weekly

### Quarter
- **Gregorian mode:** standard calendar or fiscal quarter (Q1–Q4)
- **Ethiopian mode:** group Ethiopian months in 3s —
  - Q1: Meskerem, Tikimt, Hidar
  - Q2: Tahsas, Tir, Yekatit
  - Q3: Megabit, Miyazia, Ginbot
  - Q4: Sene, Hamle, Nehase (+ Pagumē)
- Compared against: previous quarter; optionally same quarter last year
- Drill-down granularity: monthly

---

## 5. Ethiopian calendar notes for implementation
- 13 months: 12 of 30 days each, plus **Pagumē** (5 days, or 6 in an Ethiopian leap year).
- New Year (**Enkutatash**) falls on Meskerem 1, roughly **September 11** on the Gregorian calendar (September 12 in the Gregorian year immediately before a Gregorian leap year).
- The Ethiopian year runs about 7–8 years behind the Gregorian year — the exact offset depends on whether the date falls before or after Meskerem 1, so **use a maintained conversion library** (e.g., `ethiopian-date`, `kenat`, or a backend date-conversion service) rather than hardcoding the offset.
- Recommend storing all raw data in Gregorian/UTC internally and converting only at the display layer — this avoids compounding rounding/offset bugs in report math.

---

## 6. Example bot reply

> **This Week's Report** (Aug 3 – Aug 9, 2026 GC · Nehase 24–30, 2018 EC)
> Total: 1,240 (+8% vs last week)
> Top day: Wednesday (312)
> Lowest day: Sunday (98)
>
> [View Full Report]  [Export CSV]  [Compare to Last Month]

---

## 7. Edge cases to handle
- Button tapped before enough history exists → state that plainly, don't guess.
- Range crosses a timezone/DST boundary → normalize to the app's configured timezone.
- Range crosses the **Ethiopian New Year (Meskerem 1)** or into **Pagumē** → handle the short 5/6-day month correctly rather than assuming a 30-day block.
- User asks for a range outside the 4 presets → fall back to the custom date-range picker.
- Data source is slow/unavailable → show a loading state, then a clear error with a retry option — never a silent blank report.
