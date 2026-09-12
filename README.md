# PhonePe Transaction Analytics Dashboard

An end-to-end Power BI project analyzing a year (Jan–Dec 2024) of PhonePe-style transaction data — 300,000 transactions across Money Transfer, Recharge & Bills, Loans, and Insurance, for over 100,000 users. The project covers the full BI workflow: data modeling, Power Query transformations, a DAX-generated date table, DAX measures, and an interactive single-page dashboard with dynamic, auto-updating insight text.

---

## 📌 Project Overview

- **Goal:** Turn raw transaction-level data into a decision-ready dashboard that shows transaction volume, value, success/failure trends, and user behaviour patterns.
- **Data:** 2 tables, 300,000 transactions, 107,658 users, Jan–Dec 2024.
- **Tools:** Power BI Desktop, Power Query (M), DAX, Excel (source data).
- **Output:** A single main dashboard page + 2 report-page tooltips for drill-down detail.

---

## 🗂️ Dataset

Source: `Phonepe-Final-Dataset.xlsx` (2 sheets)

**All_Users** (107,658 rows)

| Column | Description |
|---|---|
| User_ID | Unique user ID (e.g. PP0000001) |
| Name | User's full name |
| Age | 18–60 |
| Join_Date | Registration date (Aug 2023 – Aug 2025) |

**All_Transactions** (300,000 rows)

| Column | Description |
|---|---|
| Transaction_ID | Unique transaction code |
| Amount | ₹20 – ₹99,999 |
| User_ID | Links to All_Users |
| Service | Money_Transfer, Recharge_Bills, Loans, Insurance |
| Service Type | Sub-category (UPI ID, Mobile Recharge, Bike Loan, Health, etc.) |
| Payment_Status | Successful, Failed, **Pending** (see Power Query step below) |
| Reason | Failure reason detail |
| Date | Jan 1 – Dec 30, 2024 |

Every `User_ID` in transactions has a matching row in `All_Users` — no orphan records.

---

## 🛠️ How I Built It

### 1. Data Modeling
Built a star schema with `All_Transactions` as the fact table and `All_Users` as a dimension, joined on `User_ID`. This keeps every DAX measure simple (no many-to-many joins) and the report fast.

### 2. Power Query — Age Segment (Conditional Column)
Added a conditional column on `All_Users.Age` to bucket raw ages into readable segments. Bucketing in Power Query (ETL stage) instead of a DAX calculated column means it's computed once at refresh time, not re-evaluated on every visual render — and it keeps the "shaping" logic separate from the "analysis" logic.

### 3. Power Query — Payment_Status Cleanup (Wrong PIN / Server error / Insufficient amount → Pending)
In Power Query, the three granular failure statuses — **Wrong PIN**, **Server error**, and **Insufficient amount** — were replaced with a single **Pending** status. This simplifies `Payment_Status` down to three clean states (Successful, Failed, Pending) for slicers and KPI cards, while the original, more granular reason is still preserved in the `Reason` column for anyone who needs to drill into *why* a transaction is pending.

### 4. DAX — Calculated Date Table
The `Date_Table` was built as a **DAX calculated table** (using `CALENDAR()` / `CALENDARAUTO()`), not in Power Query. Building it in DAX keeps it inside the model itself — it recalculates automatically off the min/max dates in `All_Transactions` — and includes Month, Month Name, Year, Quarter, and a **Weekend** flag column. This table is joined to `All_Transactions` on Date and drives every time-based slicer, trend chart, and the Weekend-vs-Weekday split, avoiding the hidden auto-date tables Power BI creates per date column.

### 5. DAX Measures
All calculations live in a dedicated `Measures` table:

| Measure | What it does |
|---|---|
| `Total Transaction Value` | SUM of Amount |
| `Total transaction` | COUNT of transactions |
| `Successful transaction` | COUNT where Payment_Status = "Successful" |
| `Success rate` | Successful ÷ Total, as % |
| `Total User` | DISTINCTCOUNT of User_ID |
| `Total trans PM` / `Trans Value PM` | Previous-month total, via DATEADD/PARALLELPERIOD on Date_Table — helper measures |
| `Total trans MOM%` / `Trans Value MoM%` | DIVIDE(Current − Previous, Previous) — feeds the growth KPI cards |
| `Calc` | Supporting helper measure used inside the dynamic text logic |

### 6. Dashboard Design (Page 1 — 22 visuals)
- **6 KPI cards** — Total Transaction Value, Total Transaction, Success rate, Total User, and two MoM% growth cards
- **Line charts** — Transaction Value & Count by Month, Success rate by Month, Total User by Month
- **Bar charts** — Value by Service, Value by top Users
- **Donut charts** — Value/Users by Age Segment, Value by Weekend vs Weekday
- **Slicers** — Month, Payment_Status
- **Branding** — PhonePe logo, custom background, calendar icon

### 7. Report-Page Tooltips
Two tooltip pages give drill-down detail without cluttering the main page:
- **Tooltip 1** — Transaction Value by Service Type (behind the Service bar chart)
- **Tooltip 2** — Transaction Value by Age Segment (behind the Age Segment donut)

### 8. Dynamic DAX-Driven Insight Text
Three text boxes use DAX measures (not static labels) to narrate the current filter context automatically:
- Which **Service** is generating the highest transaction value
- Which **Age Segment** is contributing the most
- Whether **Weekend or Weekday** drives higher total transaction value

Because these are measures, the sentences update live as the user changes the Month or Payment_Status slicer — this is the most technically interesting part of the build, since it needs a ranking function (MAXX/TOPN-style logic) that returns a *category name as text*, not just a number.

---

## 📊 Key Insights

| Metric | Value |
|---|---|
| Total Transaction Value | ₹3,474,321,934 (≈ ₹347.4 crore) |
| Total Transactions | 300,000 |
| Overall Success Rate | 96.0% |
| Total Unique Users | 107,658 (100,761 have transacted) |
| Highest value by Service | **Loans** — ₹253 crore (73% of all value), despite fewer transactions |
| Highest volume by Service | **Money_Transfer** — 150,000 transactions (50% of all volume) |
| Weekday vs Weekend value | Weekdays drive ≈ 71.6% of total transaction value |
| Highest-value age group | **36–55 age band** — contributes ≈ 46.7% of total value combined, with the highest average ticket size per transaction |
| Payment_Status split | Successful 96.0% · Failed 3.3% · **Pending** 0.7% (grouped from Wrong PIN, Server error, Insufficient amount) |

**The standout pattern:** Money_Transfer wins on *volume*, but Loans wins on *value* by a wide margin — a smaller number of high-ticket loan disbursements move far more money than a large number of small transfers. Weekdays and the mid-career age band (roughly the older-Millennial/Gen X segment) are consistently the highest-value contributors across nearly every cut of the data.

---

## 💡 Business Recommendations

Turning the insights above into concrete actions:

### 1. Loans dominate transaction value → double down on loan cross-sell
Since Loans generate the highest ₹ value despite a smaller transaction count, each loan customer is worth significantly more than an average user.
- Offer **pre-approved top-up loans** or **lower processing fees** to existing loan customers who repay on time.
- Bundle **loan protection insurance** at the point of disbursement (Insurance is already a related product — cross-sell it directly in the loan flow).
- Run **EMI cashback** or **interest-rate discount offers** for Bike/Car loan renewals, since these are the largest loan sub-categories.

### 2. Weekdays drive ~72% of value → shift marketing spend toward weekday campaigns
Weekday transactions consistently outperform weekends in both count and value.
- Schedule **push notifications, cashback offers, and loan pre-approval nudges on Tuesday–Thursday**, when engagement is naturally highest, to maximize conversion.
- Separately, run a **"Weekend Cashback" campaign** specifically to grow the *underperforming* weekend slot — e.g., extra cashback on UPI/QR transactions only on Sat–Sun to build a new habit loop, since that's the segment with the most room to grow.

### 3. The 36–55 age band spends the most → target them with premium, high-value offers
This group has both a high transaction count and the highest average ticket size — they're not just active, they're high-value.
- Push **wealth-building products** (Mutual Funds, Gold Loans) to this segment specifically, since they already show the highest engagement with Loans and Insurance.
- Introduce a **loyalty/priority tier** (e.g. "PhonePe Gold") offering faster support, higher transaction limits, or fee waivers — the kind of retention play that matters most for your highest-LTV users.
- For the **18–25 segment** (lowest value share today but the future user base), lead with smaller-ticket habit-forming offers — Mobile Recharge cashback, small-ticket UPI cashback — to build engagement before they reach higher-spending life stages.

### 4. Pending transactions need a faster resolution path
Since Wrong PIN, Server error, and Insufficient amount are now grouped as **Pending**, this bucket is effectively a "needs follow-up" queue rather than a dead end.
- Trigger an **automatic retry prompt** or SMS/push nudge for Pending transactions within a few minutes, since a chunk of these (wrong PIN, low balance) are recoverable if the user acts quickly.
- Track **Pending → Successful conversion rate** as its own KPI — it tells you how much of that 0.7% is genuinely recovered failure vs. lost revenue.
- If infra-related causes (server errors) are a meaningful share of Pending, that's a signal for a backend load review, especially around peak transaction hours.

---

## 🚀 Future Scope

- Add a dedicated **Pending-transaction drill-down view** using the `Reason` column, now that Payment_Status itself is simplified.
- Add a **map visual** if a location/state field becomes available in the source data.
- Hide helper "Previous Month" measures from the report view to keep the fields list clean.
- Fix the `Succesfull transaction` measure name (typo) and rename `Calc` to something descriptive.
- Extend dynamic insight text to also call out the **lowest**-performing category, not just the highest, for a more complete narrative.

---

## 🧰 Tech Stack

- **Power BI Desktop** — data modeling, DAX, report design
- **Power Query (M)** — Age Segment conditional column, Payment_Status cleanup
- **DAX** — calculated Date table, measures, time intelligence, dynamic text logic
- **Excel** — source dataset

---

## 📁 Files

- `Phonepe-Final-Dataset.xlsx` — source data
- `phonpe_dashboard.pbix` — Power BI report file

---

## 👨‍💻 Author

**Sandeep Dass**

Aspiring Data Analyst | Excel | Power Query | Power Pivot | DAX | SQL | Python | Power BI

* 🔗 **LinkedIn:** https://www.linkedin.com/in/sandeep-dass-a9857030a/
* 📧 **Email:** 43sandeepdas@gmail.com

---

⭐ If you found this project useful, feel free to star the repository!

