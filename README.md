# 🌍 Global Economic Outlook — Interactive Dashboard (2001–2020)

An interactive Power BI dashboard exploring 20 years of global economic data from the **International Monetary Fund (IMF) World Economic Outlook** dataset. Built to help policymakers and the general public understand patterns of economic performance at the country and country-group level.

[![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)](https://app.powerbi.com/groups/me/reports/68c0fd03-43b3-4ce8-a175-1539d2cb1cb0/c35dfa7c01014060d9a9?experience=power-bi)

**[▶ View Live Dashboard](https://app.powerbi.com/groups/me/reports/68c0fd03-43b3-4ce8-a175-1539d2cb1cb0/c35dfa7c01014060d9a9?experience=power-bi)**

---

## 📸 Dashboard Preview

![Dashboard Screenshot](dashboard/Global%20Economic%20Outlook%20(IMF%202001-2020).png)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Data Sources](#data-sources)
- [Project Structure](#project-structure)
- [Data Model](#data-model)
- [DAX Measures](#dax-measures)
- [Visualisations](#visualisations)
- [Getting Started](#getting-started)
- [Limitations & Future Work](#limitations--future-work)
- [Documentation](#documentation)

---

## Overview

This project was developed to support a thinktank scenario focused on understanding global economic development. The dataset covers **44 economic indicators** across **180+ countries** over the period **2001–2020**.

The dashboard is designed around **long-term trends** rather than single-year snapshots, providing a clearer picture of structural progress, recurring challenges, and shifts in economic conditions over time.

### Dashboard Objectives

- Highlight long-term economic trends to surface progress or instability
- Provide flexible year-range filtering for short-, medium-, and long-term analysis
- Present key indicators (GDP growth, inflation, unemployment, government debt, savings/investment) using clear and intuitive visuals
- Enable comparisons between individual countries and broader groups (regions, income classifications)
- Ensure accessibility for both analysts and non-technical audiences

---

## Key Features

- **KPI Cards with Sparklines** — Instant snapshot of six headline indicators with embedded mini trendlines and colour-coded cues (green for positive, red for negative)
- **Top 5 Bar Chart** — Dynamically ranks the best-performing countries by GDP growth, responsive to all active filters
- **Interactive Line Chart** — Tracks any selected indicator over time with a tile slicer for quick switching between GDP Growth, Inflation, Unemployment, Fiscal Balance, Government Debt, and Current Account Balance
- **Radar Chart** — Multi-indicator economic profile for comparing performance across dimensions at a glance
- **Flexible Slicers** — Filter by Country, Region, Income Group, and Year Range — every visual responds instantly

---

## Data Sources

All data originates from the **IMF World Economic Outlook** database.

| File | Description |
|------|-------------|
| `Data.xlsx` | Country-level indicator values by year (WEO country code, ISO, country name, year, 44 indicators) |
| `Country Groupings.xlsx` | Country metadata — country code, region, and income group classification |
| `Metadata.xlsx` | Indicator definitions — indicator code, description, and unit of measurement |

---

## Project Structure

```
imf-globaleconomy-analytics/
│
├── dashboard/
│   ├── Global Economic Outlook (IMF 2001-2020).pbix   # Power BI report file
│   └── liveurl-dashboard.txt                          # Live dashboard URL
│
├── dax-measures/
│   ├── card-measures.txt                              # DAX for KPI card callout values & sparklines
│   ├── line-chart-measures.txt                        # DAX for interactive line chart & field parameter
│   └── radar-chart-measures.txt                       # DAX for radar table & dynamic indicator values
│
├── report-documentation/
│   └── Global Economic Outlook — Interactive Dashboard Report (2001-2020).pdf
│
├── source-data/
│   ├── Data.xlsx
│   ├── Country Groupings.xlsx
│   └── Metadata.xlsx
│
└── README.md
```

---

## Data Model

The data model follows a **star schema** with one fact table and two dimension tables:

```
┌──────────────────┐       ┌──────────────────┐
│   Countries       │       │   Indicators      │
│ (Dimension)       │       │ (Dimension)       │
├──────────────────┤       ├──────────────────┤
│ CountryKey (PK)   │       │ Indicator Code    │
│ ISO               │◄──┐   │ Description       │
│ Country           │   │   └────────┬─────────┘
│ Region            │   │            │
│ Income Group      │   │            │ 1:* (Description → Indicator)
└──────────────────┘   │            │
                        │   ┌────────▼─────────┐
              1:* (ISO) └───┤  Economic Data     │
                            │  (Fact)            │
                            ├──────────────────┤
                            │ Country           │
                            │ ISO               │
                            │ Year              │
                            │ Indicator         │
                            │ Indicator Value   │
                            │ WEO Country Code  │
                            └──────────────────┘
```

### Key Preparation Steps

1. **Loaded** all three Excel datasets via Power Query
2. **Removed duplicates** and renamed tables (`Economic Data`, `Countries`, `Indicators`)
3. **Added a CountryKey** index column to the Countries table for effective cross-filtering
4. **Unpivoted** the fact table — transformed wide-format indicator columns into rows (`Indicator`, `Indicator Value`)
5. **Merged Description + Units** in the Indicators table into a single `Description` column to match the unpivoted fact table
6. **Created relationships** — ISO (1:*) and Description → Indicator (1:*)

---

## DAX Measures

### Card Measures (KPI Callout Values & Sparklines)

Each indicator has two measures — one for the card callout (divided by 100 for percentage formatting) and one for the per-year sparkline:

```dax
GDPGrowth =
CALCULATE(
    DIVIDE(AVERAGE('Economic Data'[Indicator Value]), 100),
    KEEPFILTERS('Economic Data'[Indicator] =
        "Gross domestic product, constant prices,Percent change")
)

GDPGrowth_PerYear =
CALCULATE(
    AVERAGE('Economic Data'[Indicator Value]),
    'Economic Data'[Indicator] = "Gross domestic product, constant prices,Percent change"
)
```

Similar pairs exist for **Inflation**, **Unemployment**, **Government Debt**, **Fiscal Balance**, and **Current Account Balance**.

### Radar Chart Measures

A calculated table provides category labels, and a SWITCH measure dynamically returns the correct value:

```dax
RadarTable =
DATATABLE("Indicator", STRING,
    {{"GDP Growth"}, {"Inflation"}, {"Unemployment"},
     {"Fiscal Balance"}, {"Debt"}, {"Current Account"}})

Indicator Value =
VAR SelectedIndicator = SELECTEDVALUE(RadarTable[Indicator])
RETURN SWITCH(
    SelectedIndicator,
    "GDP Growth",       [GDPGrowth],
    "Inflation",        [Inflation_rate],
    "Unemployment",     [Unemployment_rate],
    "Fiscal Balance",   [Fiscal_pct],
    "Debt",             [GovtDebt_pct],
    "Current Account",  [CABal_pct],
    BLANK())
```

### Line Chart Measures

A field parameter enables dynamic indicator switching via a tile slicer:

```dax
Line Trend Indicators = {
    ("GDP Growth", NAMEOF('Economic Data'[GDPGrowth]), 0),
    ("Current Account Balance", NAMEOF('Economic Data'[CABal_pct]), 1),
    ("Fiscal Balance", NAMEOF('Economic Data'[Fiscal_pct]), 2),
    ("Government Debt", NAMEOF('Economic Data'[GovtDebt_pct]), 3),
    ("Inflation Rate", NAMEOF('Economic Data'[Inflation_rate]), 4),
    ("Unemployment Rate", NAMEOF('Economic Data'[Unemployment_rate]), 5)
}
```

---

## Visualisations

| Visual | Purpose | Key Design Choice |
|--------|---------|-------------------|
| **KPI Cards + Sparklines** | At-a-glance headline indicators | Colour-coded trendlines (green = positive, red = negative) |
| **Clustered Bar Chart** | Top 5 countries by GDP growth | Top N filter dynamically responds to slicers |
| **Line Chart** | Year-over-year trend for selected indicator | Tile slicer for switching indicators without page clutter |
| **Radar Chart** | Multi-dimensional economic profile | Enables balanced comparison across 6 indicators simultaneously |
| **Slicers** | Country, Region, Income Group, Year Range | Dropdown and Between styles to save canvas space |

---

## Getting Started

### Prerequisites

- [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (free)

### Local Setup

1. Clone or download this repository
2. Open `dashboard/Global Economic Outlook (IMF 2001-2020).pbix` in Power BI Desktop
3. If prompted, point the data source paths to the files in `source-data/`
4. Interact with the slicers and visuals to explore the data

### Or View Online

**[Open the live dashboard →](https://app.powerbi.com/groups/me/reports/68c0fd03-43b3-4ce8-a175-1539d2cb1cb0/c35dfa7c01014060d9a9?experience=power-bi)**

---

## Limitations & Future Work

### Current Limitations

- Averaged values can smooth out important year-to-year volatility
- Data gaps in the IMF dataset may affect accuracy for certain countries in earlier years
- The radar chart simplifies complex economic relationships and may overstate differences at small scales
- The dashboard is **descriptive** — it highlights trends but does not explain underlying causes or policy effects

### Potential Enhancements

- Add **forecasting** visuals using Power BI's built-in analytics or external ML models
- Incorporate **drill-through pages** for country-level deep dives
- Add **tooltips with contextual narratives** for major economic events (e.g., 2008 financial crisis, COVID-19)
- Integrate additional data sources (World Bank, UN) for a richer cross-referenced view
- Build a **decomposition tree** to explore drivers of indicator changes

---

## Documentation

The full project report — including design rationale, data preparation steps, DAX formulas, and critical evaluation — is available in:

📄 [`report-documentation/Global Economic Outlook — Interactive Dashboard Report (2001-2020).pdf`](report-documentation/)

---

## License

This project uses publicly available data from the [IMF World Economic Outlook Database](https://www.imf.org/en/Publications/WEO). The dashboard and analysis are provided for educational and analytical purposes.