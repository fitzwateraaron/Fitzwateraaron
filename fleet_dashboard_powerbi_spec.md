# Live Fleet Maintenance Dashboard (Power BI + OneDrive Excel)

Use this as an implementation brief for building a **live fleet maintenance dashboard** for a landscape supply company.

## 1) Data source

Connect Power BI to an Excel 365 workbook stored in OneDrive/SharePoint with these tables:

- `tblAssets` (sheet: **ASSETS**)
- `tblLog` (sheet: **MASTER_LOG**)
- `tblSchedule` (sheet: **SCHEDULE**)

Assume all are formatted as Excel tables with stable, consistent column names.

---

## 2) Scope and goal

Build a **Total Fleet** dashboard focused on maintenance operations (not taxes), with:

- Executives/managers able to filter by date, asset, and work categories.
- Mechanics/operations able to see due PM work and current down assets.
- Mobile-friendly consumption in Power BI Service.

---

## 3) Power BI build requirements

### Filters (slicers)

- Log date range
- `AssetType`
- `AssetID`
- `WorkType`
- `SystemArea`
- `Down` (Yes/No)

### KPI cards

- Total maintenance cost (selected period)
- YTD maintenance cost
- Work entry count (selected period)
- Assets currently down (latest known status per asset)
- PM items due now
- PM items due soon

### Visuals

- Cost by month (last 12 months) — line chart
- Top 10 assets by cost — bar chart
- Cost by `SystemArea` — bar or donut
- `WorkType` mix — bar or donut
- Due / Due Soon table from `tblSchedule` sortable by `NextDueDate` / `NextDueMeter`
- Recent work table (last 30 rows): `AssetID`, date, system area, description, cost

### Drill-through

Selecting an `AssetID` should open an **Asset Detail** page with:

- Asset profile from `tblAssets` (VIN, plate, make/model/year)
- Full work history table
- Cost trend (time series) for selected asset
- Most common `SystemArea` for selected asset
- PM schedule status for selected asset

---

## 4) Suggested data model

- `tblAssets` (1) -> (many) `tblLog` on `AssetID`
- `tblAssets` (1) -> (many) `tblSchedule` on `AssetID`
- Calendar table (marked as Date table) related to `tblLog[LogDate]`

Recommended helper columns:

- `tblLog[Cost]` numeric (currency)
- `tblLog[DownFlag]` boolean if source uses text
- `tblSchedule[DueBucket]` calculated column (`Due`, `Due Soon`, `Not Due`)

---

## 5) DAX measure starter pack

> Rename fields as needed to match your workbook schema.

```DAX
Total Maintenance Cost =
SUM(tblLog[Cost])

YTD Maintenance Cost =
TOTALYTD([Total Maintenance Cost], 'Calendar'[Date])

Work Entries =
COUNTROWS(tblLog)

PM Due =
CALCULATE(
    COUNTROWS(tblSchedule),
    tblSchedule[DueBucket] = "Due"
)

PM Due Soon =
CALCULATE(
    COUNTROWS(tblSchedule),
    tblSchedule[DueBucket] = "Due Soon"
)

Recent Work Count (30) =
VAR RankedRows =
    TOPN(
        30,
        ALLSELECTED(tblLog),
        tblLog[LogDate], DESC,
        tblLog[AssetID], ASC
    )
RETURN
COUNTROWS(RankedRows)
```

### Latest-status down-asset pattern

Create a summarized latest-status table in Power Query or with DAX logic to evaluate **latest log row per asset**, then count assets where latest `Down=Yes`.

---

## 6) Refresh and publishing

1. Save workbook in OneDrive/SharePoint (org tenant).
2. In Power BI Desktop: **Get Data -> Web/SharePoint/OneDrive** and connect to workbook.
3. Build model, measures, report pages, slicers, drill-through.
4. Publish to Power BI Service workspace.
5. Configure dataset credentials and **Scheduled refresh**.
6. Pin visuals/KPIs to a dashboard for mobile app consumption.
7. Share with org users via workspace/app (license-dependent).

---

## 7) Licensing guidance

- **Power BI Pro available:** easiest path for sharing + scheduled refresh.
- **No Pro / Free only:** dashboard sharing is constrained; evaluate:
  - upgrading to Pro (recommended), or
  - alternative app stack (e.g., lightweight web app with hosted DB/API).

---

## 8) Deliverables checklist

- `.pbix` file containing model, measures, visuals, and drill-through pages
- Setup runbook with:
  - OneDrive connection steps
  - publish steps
  - refresh configuration
  - sharing and access model
- DAX measure list (final names and formulas)
