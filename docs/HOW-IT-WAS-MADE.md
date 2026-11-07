# How the MyMoney prototype was made

A beginner's guide to `index.html`. It's built the same way as park4u, so if you've read that guide, most of this will feel familiar. Line numbers point into `index.html` (in VS Code press `Ctrl+G` to jump to a line).

---

## 1. One file, three languages

| Language | What it does | Where |
|---|---|---|
| **HTML** | the things on screen | the `<section>` blocks (line ~245) |
| **CSS** | how they look | inside `<style>` (lines 13–240) |
| **JavaScript** | what they do | inside `<script>` (from line 276) |

There's no server and no database. Your data is saved in the browser's **localStorage**, a small storage box each website gets. Close the tab and come back: your data is still there.

---

## 2. It started from your first MyMoney file

I kept everything from `MyMoney – Personal Finance.html`:

* the same **categories**: 8 for expenses, 3 for income
* the same **colours**: blue `#0d5bd7`, green for income, red for spending, amber for warnings, plus the dark-mode colours
* the same **Home** layout: month bar → balance → spending donut → last 6 months → transactions
* the same **Budgets** screen, with the bar turning amber at 80% and red when you go over
* the "**Saved**" toast and the **mark paid → logs an expense** rule for bills

**One change:** the old file coloured a category by where it ranked on the donut, so Food could be blue one month and red the next. Now each category always keeps one colour (`EXP_INFO`, line ~302).

**Your old data comes along:** on first launch, if the browser still has the first prototype's data (key `mymoney1`), the welcome screen offers to bring it in (`importOld`, line 361).

---

## 3. The iPhone look (borrowed from park4u)

* **Phone frame**: `.phone` (line 39), with the status bar, Dynamic Island and home bar. On a real phone (screen narrower than 500px) the frame disappears and the app fills the screen (`@media`, line 46).
* **Large titles** like iOS apps (`.lt`), **grouped lists** like the Settings app (`.group` / `.cell`), **segmented controls** (`.seg`), **switches** (`.switch`).
* **Tab bar** at the bottom: Home · Transactions · Bills · Goals · Insights (`TABS`, line 464).
* **Sheets** slide up for forms (`openSheet`, line 507), the **action sheet** pops up for choices (`actionSheet`, line 495), and **banners** drop down like iPhone notifications.

---

## 4. Screens

| Screen | What you can do |
|---|---|
| **Welcome** | enter your name, start fresh or try sample data |
| **Home** | month switcher, balance, bill reminders with a Pay button, donut, 6-month bars, budgets, recent items |
| **Transactions** | search, Expenses/Income, this month or all time, filter by category and payment method; tap a row to edit or delete |
| **New transaction** (sheet) | amount, category, UPI/Cash/Card/Bank, date, note, optional receipt photo |
| **Budgets** | type a monthly limit for each category |
| **Bills → Bills & rent** | add bills or rent with a due day 1–31, mark paid, undo, edit, delete |
| **Bills → Roommates** | add roommates, split an expense, see who owes whom, settle up |
| **Goals** + goal page | progress ring, "save ₹X a month to reach it by…", add money, withdraw, history |
| **Insights** | biggest increase, budget pace, average daily spend, month-end projection, saving tip |
| **Settings** | name, currency, theme, notification switches, CSV export, backup / restore, sample data, delete everything |

---

## 5. How the JavaScript is organised

The script is split into numbered sections. Search for `====` to jump between them.

1. **Icons**: small SVG drawings.
2. **Categories**: names, colours, emoji.
3. **Helpers**: dates, money formatting (`fmt`), `esc` (stops typed text from breaking the page).
4. **Data**: `state` holds *everything*; `save()` writes it to localStorage; `loadSample()` makes 6 months of realistic Kochi data.
5. **Calculations**: totals, bill status, roommate balances.
6. **Navigation**: `tab()` switches tabs, `push()` slides a screen in, and the phone's back button works through `history`.
7. **Overlays**: toast, banner, action sheet, form sheet.
8. **Alerts**: reminders.
9. **Shared UI**: the month bar, transaction rows, donut and bars.
10. **Screens**: one `RENDER.<name>` function per screen. Each one builds HTML text and puts it on the page.
11. **Forms**: add/edit transaction, bill, split, goal.
12. **Export**: CSV and JSON downloads.
13. **Taps**: one click listener for the whole app (explained below).
14. **Start**: apply the theme, then show Welcome or Home.

### The one-listener trick

Instead of wiring up every button separately, buttons carry a note:

```html
<button data-act="payBill" data-id="b2">Pay</button>
```

and one listener (line ~950) looks up `ACT.payBill` and calls it with `"b2"`. To add a new button you write the HTML with a `data-act` and add one function to `ACT`.

### Re-draw, don't patch

After any change (add, pay, delete) the code calls `save()` and then `refresh()`, which runs the current screen's `RENDER` function again. Drawing the whole screen from `state` every time is simple and hard to get wrong.

---

## 6. The tricky logic, explained

**Bills due on the 31st** (`dueDate`, line ~440):
```js
const dueDate = (b, m) => m + '-' + pad(Math.min(b.day, daysIn(m)));
```
`Math.min` picks the smaller number. In February `daysIn` is 28 (or 29), so day 31 becomes the 28th. Each bill keeps a list of the months it was paid (`paid: ['2026-10', '2026-11']`), so when a new month starts the bill is simply "not paid yet". That's the month rollover.

**Reminders** (`checkAlerts`, line 514): when the app opens, it looks for bills due within 2 days, due today or overdue, and budgets at 80% or 100%. Each reminder is remembered in `state.notified` so it never shows twice.

**Who owes whom** (`mateBalance`, line 451): for every shared expense, each person's share is `amount ÷ number of people`. If you paid, they owe you their share. If they paid, you owe them yours. Settle-ups cancel it out.

**Insights**: "biggest increase" compares this month with the **same number of days** of last month, so a half-finished month is compared fairly.

---

## 7. Run it

Double-click `index.html` (it opens in your browser), or in VS Code right-click → *Open with Live Server*.
On your phone, put it on GitHub Pages and open the link in Safari or Chrome, then use *Add to Home Screen*.

## 8. Ideas to try yourself

* Add a new expense category (add it to `EXP` and `EXP_INFO`).
* Show the receipt photo bigger when you tap it.
* Add a "Monthly report" card that shows the top 3 categories.
