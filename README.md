# Female Primary Education Completion Analysis

This repository contains a Power BI analysis of female primary education completion rates across countries and income groups, using World Bank data for 2020 to 2024. The project focuses on comparing completion patterns over time, highlighting countries with large gains or declines, and identifying differences by region and income level.

## Repository contents

- `Cleaned_data_female_primary_completion_2020-2024.csv` – cleaned dataset used for the analysis.
- `Female Primary Education Completion Analysis (2020 to 2024).pbix` – the Power BI report file.
- `API_SE.PRM.CMPT.FE.ZS_DS2_en_csv_v2_10665 (1).zip` – original World Bank download archive.
- `README.md` – overview and reproduction guide.
- `docs/dax.md` – DAX definitions used in the report.

## Objective

The dashboard was designed to answer three questions:

1. How has female primary education completion changed between 2020 and 2024?
2. Which countries improved most, and which declined most?
3. How do completion rates differ by income group and region?

## Data source

The underlying data is based on the World Bank indicator for female primary completion rate. In World Bank wording this is often represented as:

- `SE.PRM.CMPT.FE.ZS`
- Female primary completion rate (% of relevant age group)

The raw source data was downloaded as a zipped CSV from the World Bank API and then cleaned for analysis.

Important caveat:

- Some values exceed 100%. This can happen in reported education statistics because of differences in denominators, reporting conventions, or data quality issues.
- These values should be interpreted as reported values, not as a guarantee that completion exceeds 100% of a cohort.
- For the business narrative, it is best to treat them as data quality / reporting anomalies and avoid over-interpreting extreme values.

## Dataset structure

The cleaned file contains these columns:

- `Country`
- `Region`
- `Income Group`
- `Year`
- `Average of Completion Rate`
- `Reporting Countries`

Example rows look like this:

```csv
Country,Region,Income Group,Year,Average of Completion Rate,Reporting Countries
Afghanistan,Middle East & North Africa,Low income,2023,76.6787,1
Albania,Europe & Central Asia,Upper middle income,2020,97.0401,1
```

## Quick analysis summary

This summary reflects the cleaned dataset and is intended as a narrative summary of the report, not a replacement for the Power BI model:

- Average completion among reporting countries was about 92.88% in 2020 and 93.40% in 2024.
- That indicates a modest increase of roughly 0.52 percentage points over the four-year period.
- High-income countries generally reported much higher completion than low-income countries.
- There were some large country-specific declines, which are worth investigating because they may reflect conflict, data gaps, policy disruption, or reporting inconsistency.
- Countries such as Niger showed a sustained decline across the period and warrant close attention.

## Reproducing the analysis in Power BI

### Prerequisites

- Power BI Desktop
- The cleaned CSV file in this repository

### Step 1: Import the data

1. Open Power BI Desktop.
2. Click `Get Data`.
3. Select `Text/CSV`.
4. Choose `Cleaned_data_female_primary_completion_2020-2024.csv`.
5. Load the file into the model.

### Step 2: Clean and format

In Power Query, check that the column names and data types are correct:

- `Country` = Text
- `Region` = Text
- `Income Group` = Text
- `Year` = Whole Number
- `Average of Completion Rate` = Decimal Number
- `Reporting Countries` = Whole Number

Rename the data field consistently if needed. The report appears to use a table named `Female_Completion`, so if you rebuild the measures in DAX, define your table to match that name.

### Step 3: Build the model

Use a star-schema style model with a single fact table containing country-level completion records. You can create slicers for:

- `Year`
- `Region`
- `Income Group`

A simple model is enough for this report because there are no complex relationships required.

### Step 4: Add core measures

Create the measures listed in `docs/dax.md`.

At minimum, the report relies on:

- `Average Completion Rate`
- `Matched Country Change (pp)`
- `Matched Country Count`
- `Change Rank`
- `Change Bar Colour`

### Step 5: Create visuals

Suggested visuals:

- KPI card for average completion rate in 2020 and 2024
- Matrix or table for country rankings by improvement/decline
- Bar chart for top positive and negative changes
- Map visual by country
- Slicer for region and income group

### Step 6: Validate findings

Before publishing or presenting, confirm:

- the comparison year is correctly filtered to 2020 and 2024
- the `Matched Country Change (pp)` logic is applied only to countries with both years present
- the ranking logic is using the same measure across all countries
- unusual values above 100% are not being treated as literal completion above 100% without context

## Interpretation guidance

This analysis is strongest when framed as a descriptive cross-country comparison rather than a causal proof of policy effectiveness. The data is useful for identifying patterns, but it cannot by itself explain why some countries improved or declined. For example:

- conflict and displacement can disrupt schooling
- data gaps or reporting changes can distort trends
- funding, infrastructure, and access to schools affect completion
- a country can have a high average while still having large gaps within a population

## Notes on the findings

Some of the original narrative in the project description is valid, but a few points should be handled carefully:

- The phrase “High income countries performed 29.26% better” is best expressed as a percentage-point difference or as a comparison of average completion rates, not as a raw percentage increase.
- “Seychelles saw a change of -39.65” is a large drop and should be identified as a notable anomaly or data-quality concern unless validated against source data.
- “Niger declined by ~14.0 percentage points over four years” is a sobering result and should be presented with context, not as a universal trend for all low-income countries.

## How to use this repository

If you want to reproduce the analysis exactly:

1. Open the `.pbix` file in Power BI Desktop.
2. Confirm data source points to `Cleaned_data_female_primary_completion_2020-2024.csv`.
3. Refresh the model.
4. Validate the DAX measures from `docs/dax.md`.
5. Publish or export the report as needed.

If you want to rebuild it from scratch:

1. Start with the CSV file.
2. Clean the field names and types.
3. Add the DAX measures.
4. Design the visuals from the summary findings above.

## Recommended reporting wording

For presentation and documentation purposes, the analysis is best described as:

> This dashboard compares female primary education completion rates across countries between 2020 and 2024 using World Bank data. It highlights country-level changes, regional and income-group differences, and countries with the most significant gains or declines in reported completion rates.

This wording is clearer and less likely to overstate what the data can prove.

## License and usage

This repository is intended for educational and analytical use. The dataset is based on World Bank public data, and the project documentation is a companion for the report rather than an official statistical publication.

---

If you want, I can also help you add:

- a more polished executive summary page inside the `.pbix`
- a slide-style narrative for presentation
- a cleaned Power BI data dictionary
- a `docs/analysis.md` file with narrative findings and chart explanations
