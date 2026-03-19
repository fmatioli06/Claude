---
name: power-bi
description: Use when the user asks about Power BI — building reports, dashboards, data models, DAX formulas, measures, calculated columns, Power Query (M), data sources, relationships, or troubleshooting Power BI issues.
---

# Power BI Skill

Help users design, build, and troubleshoot Power BI solutions — including data modeling, DAX, Power Query, and report/dashboard design.

## Workflow

Make a todo list for all tasks in this workflow and work through them one at a time.

### 1. Understand the Goal

Ask clarifying questions if needed:
- What is the business question or KPI being measured?
- What data sources are involved (SQL, Excel, SharePoint, API, etc.)?
- Is this a new report/model or modifying an existing one?
- What is the target audience (executive dashboard, operational report, self-service)?

### 2. Data Modeling

Follow star schema best practices:
- Separate **fact tables** (transactions, events) from **dimension tables** (dates, customers, products)
- Create a dedicated **Date table** marked as the official date table
- Define relationships with correct cardinality (1:many preferred) and cross-filter direction
- Avoid bidirectional relationships unless explicitly required
- Hide foreign key columns from report view

**Date table template (DAX):**
```dax
Date =
ADDCOLUMNS(
    CALENDAR(DATE(2020,1,1), DATE(2030,12,31)),
    "Year", YEAR([Date]),
    "Quarter", "Q" & QUARTER([Date]),
    "Month", FORMAT([Date], "MMMM"),
    "Month Number", MONTH([Date]),
    "Week Number", WEEKNUM([Date]),
    "Day of Week", FORMAT([Date], "dddd"),
    "Is Weekday", IF(WEEKDAY([Date], 2) <= 5, TRUE, FALSE)
)
```

### 3. DAX Measures

Write clean, performant DAX:
- Always use **explicit measures** over implicit aggregations
- Use `CALCULATE` to modify filter context
- Prefer `DIVIDE(numerator, denominator, 0)` over `/` to avoid division-by-zero errors
- Use variables (`VAR`) to improve readability and avoid repeated evaluation
- Add format strings to all measures

**Common patterns:**
```dax
-- Basic aggregation
Total Sales = SUM(Sales[Amount])

-- Safe division
Profit Margin % =
DIVIDE(
    [Total Profit],
    [Total Sales],
    0
)

-- Year-over-year comparison
Sales YoY % =
VAR CurrentSales = [Total Sales]
VAR PriorSales = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))
RETURN
DIVIDE(CurrentSales - PriorSales, PriorSales, 0)

-- Running total
Running Total Sales =
CALCULATE(
    [Total Sales],
    FILTER(
        ALL('Date'[Date]),
        'Date'[Date] <= MAX('Date'[Date])
    )
)
```

### 4. Power Query (M)

Write efficient M transformations:
- Apply filters and column selections **early** to reduce data volume
- Use **query folding** where possible (SQL sources)
- Parameterize data source connections
- Document steps with descriptive names

**Common patterns:**
```m
// Remove nulls and filter
#"Filtered Rows" = Table.SelectRows(Source, each [Status] = "Active" and [Amount] <> null),

// Add custom column
#"Added Custom" = Table.AddColumn(#"Filtered Rows", "Full Name",
    each [First Name] & " " & [Last Name], type text),

// Unpivot columns
#"Unpivoted" = Table.UnpivotOtherColumns(#"Added Custom", {"ID", "Date"}, "Attribute", "Value")
```

### 5. Report & Dashboard Design

Follow UX best practices:
- Use a consistent color palette (align with brand kit if provided)
- Place the most important KPI at the top-left (reading order)
- Use card visuals for single KPIs, bar/column for comparisons, line for trends
- Add slicers for Date, Region, Category — sync slicers across pages
- Avoid 3D charts, excessive colors, and chart junk
- Add tooltips and drill-through pages for detail
- Test on both desktop and mobile layout

**Recommended visual types by use case:**
| Goal | Visual |
|---|---|
| Single KPI | Card / KPI visual |
| Compare categories | Clustered bar / column |
| Show trend over time | Line chart |
| Part-to-whole | Donut / Stacked bar |
| Correlation | Scatter plot |
| Geographic data | Map / Filled map |
| Detail table | Matrix / Table |

### 6. Performance Optimization

- Use **Import mode** by default; use DirectQuery only when real-time data is required
- Reduce table columns to only what is needed
- Avoid calculated columns when a measure can do the job
- Use **aggregation tables** for large fact tables
- Check performance with Performance Analyzer (View > Performance Analyzer)
- Avoid `FILTER(ALL(...))` patterns when `ALLEXCEPT` or `REMOVEFILTERS` suffice

### 7. Security (Row-Level Security)

```dax
-- Dynamic RLS (filter by logged-in user's email)
[Email] = USERPRINCIPALNAME()
```

- Define roles in Model > Manage Roles
- Test roles with "View as" before publishing
- Document role membership requirements for IT/admin

### 8. Publishing & Sharing

- Publish to correct **Workspace** (avoid My Workspace for shared reports)
- Set **scheduled refresh** if using Import mode
- Configure **gateway** for on-premise data sources
- Set appropriate **permissions** (Viewer, Contributor, Member, Admin)
- Enable **sensitivity labels** for confidential data

## Wrap Up

Summarize what was built or advised:
- Data model changes made
- DAX measures created or fixed
- Power Query transformations applied
- Report/dashboard design decisions
- Any performance or security recommendations
- Next steps for the user
