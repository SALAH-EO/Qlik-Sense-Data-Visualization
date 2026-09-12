# Qlik Sense Palestine Fatalities Analytics

## Overview

The **Qlik Sense Palestine Fatalities Analytics** project is an enterprise-grade Business Intelligence (BI) and data visualization solution designed to ingest, transform, and model complex casualty datasets. Built using **Qlik Sense**, this application translates multi-decade longitudinal data into interactive, executive-level analytics, enabling data-driven humanitarian insights, trend analysis, and demographic breakdowns.

Designed with BI best practices in mind, this repository demonstrates end-to-end data engineering, multi-tier QVD architecture, advanced data modeling (Star Schema), set analysis expression engineering, and user-centric dashboard design tailored for analysts, researchers, and recruiters evaluating senior BI engineering capability.

## Architecture

The project follows a robust **3-Tier QVD Architecture** to optimize reload performance, ensure data governance, and maintain scalability across enterprise environments.

```
[ Raw CSV / API Data ]
        │
        ▼
┌─────────────────────────┐
│  Extract Layer (QVD 1)  │  --> Native, un-transformed QVD extraction
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ Transform Layer (QVD 2) │  --> Data cleansing, master calendar generation,
└───────────┬─────────────┘      demographic categorization, lookup keys
            │
            ▼
┌─────────────────────────┐
│ Data Model Layer (QVD 3)│  --> Optimized Star Schema, Synthetic Key resolution,
└───────────┬─────────────┘      Circular Reference elimination
            │
            ▼
┌─────────────────────────┐
│ Executive Dashboards    │  --> Qlik Sense UI (Set Analysis, Dynamic KPI Objects)
└─────────────────────────┘

```

1. **Extract Layer (Layer 1):** Ingests raw structured data (CSV/Excel/REST API) directly into native QVD files with zero transformation to preserve source integrity and minimize database lock times.

2. **Transform Layer (Layer 2):** Performs data standardization, handles null values, enriches spatial attributes, builds custom flags (e.g., minor/adult categorizations), and generates a continuous **Master Calendar**.

3. **Data Model Layer (Layer 3):** Loads optimized QVDs into the Qlik memory engine structured in a clean Star Schema, eliminating circular references and synthetic keys.

## Key Features

* **Multi-Dimensional Demographic Analysis:** Granular breakdowns by age bracket, gender, citizenship, region, and event context.

* **Geospatial Intelligence:** Interactive map visualizations categorizing incidents across regions (West Bank, Gaza Strip, Israel) using custom point and area layers.

* **Dynamic Time-Series Analytics:** Flexible granularity toggles (Year, Quarter, Month, Event-based timeline) using dynamic Qlik variables.

* **Enterprise Governance:** Strict implementation of dynamic data modeling, consistent color palette standards, and optimized load scripts.

* **Advanced KPI KPI Engine:** High-performance expression calculation utilizing Qlik's internal memory engine for sub-second user responsiveness.

## Technical Implementation

### Data Model Design

The data model uses a central **Fact Table** (`FactFatalities`) linked to dedicated **Dimension Tables** (`DimCalendar`, `DimDemographics`, `DimLocation`, `DimEventDetails`).

```
                      ┌──────────────────────┐
                      │     DimCalendar      │
                      └──────────┬───────────┘
                                 │ DateKey
                                 ▼
┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│  DimDemographics ├───>│  FactFatalities  │<───┤   DimLocation    │
└──────────────────┘    └──────────┬───────┘    └──────────────────┘
                              DemKey │ LocKey
                                     ▼
                        ┌──────────────────────────┐
                        │     DimEventDetails      │
                        └──────────────────────────┘

```

### Data Cleansing & Quality Assurance

* Automated handling of missing dates and age values via conditional script logic (`Coalesce()`, `Alt()`).

* Standardization of geographic spellings to support map layer matching.

* Implementation of `DISCONNECT` patterns where non-associative isolated tables are required for dynamic parameter selection.

## Dashboard Visualizations

The Qlik Sense application features four core analytical sheets:

1. **Executive Overview:** High-level metrics showing total fatalities, monthly trend lines, demographic distribution, and spatial heatmaps.

2. **Temporal & Trend Analysis:** Time-series charts analyzing monthly spikes, year-over-year percentage shifts, and cumulative fatality rates.

3. **Demographic Breakdown:** Deep-dive analysis using stacked bar charts and pivot tables focusing on age distribution, gender ratios, and vulnerability indicators.

4. **Geospatial & Incident Profiling:** Detailed spatial maps paired with tabular incident logs for detailed auditability and research verification.

## Advanced Qlik Scripting

Here are representative snippets from the data reload script showcasing enterprise Qlik scripting techniques:

### 1. Automated Master Calendar Generation

```
// Establish minimum and maximum dates from Fact data
TempCalendar:
LOAD 
    Min(EventDate) as MinDate,
    Max(EventDate) as MaxDate
RESIDENT FactFatalities;

LET vMinDate = Num(Peek('MinDate', 0, 'TempCalendar'));
LET vMaxDate = Num(Peek('MaxDate', 0, 'TempCalendar'));

DROP TABLE TempCalendar;

// Generate continuous date range
MasterCalendar:
LOAD
    TempDate as EventDate,
    Date(TempDate, 'YYYY-MM-DD') as FormattedDate,
    Year(TempDate) as Year,
    Month(TempDate) as Month,
    YearToDate(TempDate) * -1 as YTD_Flag,
    Dual(Year(TempDate) & '-Q' & Ceil(Month(TempDate)/3), Ceil(Month(TempDate)/3)) as YearQuarter,
    Week(TempDate) as Week
INLINE []; // Generated via IterNo()

MasterCalendarGenerator:
LOAD 
    Date($(vMinDate) + IterNo() - 1) as TempDate
AUTOGENERATE 1
WHILE $(vMinDate) + IterNo() - 1 <= $(vMaxDate);

```

### 2. Complex Set Analysis Expressions

Used for dynamic KPI expressions without requiring data model duplication:

* **YTD Fatalities Comparison:**

  ```
  Sum({<Year = {$(=Max(Year))}, YTD_Flag = {1}>} FatalityCount)
  
  ```

* **Targeted Regional Filter (Excluding Nulls/Unknowns):**

  ```
  Sum({<Region = {'Gaza Strip', 'West Bank'}, AgeGroup = -{'Unknown'}>} FatalityCount)
  
  ```

* **Prior Year Same Period (PYTD):**

  ```
  Sum({<Year = {$(=Max(Year)-1)}, Month = {"<=$(#vMaxMonth)"}>} FatalityCount)
  
  ```

## How to Run

### Prerequisites

* **Qlik Sense Desktop** (November 2022 release or newer) **OR** access to a **Qlik Sense Enterprise SaaS / Windows** environment.

* Git installed on your local machine.

### Installation & Execution Steps

1. **Clone the Repository:**

   ```
   git clone https://github.com/your-username/qlik-palestine-fatalities-analytics.git
   cd qlik-palestine-fatalities-analytics
   
   ```

2. **Data Directory Setup:**

   * Ensure source CSV files are placed in the `./Data/Source/` folder.

   * Set up standard Qlik lib connection paths pointing to local folders:

     * `lib://DataFiles/Extract/`

     * `lib://DataFiles/Transform/`

3. **Import App to Qlik Sense:**

   * Copy the `.qvf` file located in the `/App/` folder to your Qlik Sense apps folder:
     `C:\Users\<Your-User>\Documents\Qlik\Sense\Apps\`

   * Open **Qlik Sense Desktop** (or import via **QMC** in Enterprise environments).

4. **Execute Reload:**

   * Open the app in Qlik Sense.

   * Navigate to the **Data Load Editor**.

   * Click **Reload Data** to execute the 3-Tier QVD ETL script and populate the application.

## Technical Stack

* **BI Engine:** Qlik Sense Engine (QIX)

* **Scripting:** Native Qlik Script Language, Advanced Set Analysis

* **Data Modeling:** Star Schema Optimization, QVD Architecture

* **Source Data:** Structured CSV / Public Humanitarian Datasets

  ---

  # 👨‍💻 Author

  **Salah Eddine Ouirra**
