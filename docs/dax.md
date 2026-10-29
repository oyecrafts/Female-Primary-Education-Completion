# DAX measures for the female primary completion report

This page documents the DAX used in the Power BI report for the female primary education completion analysis.

## Assumed table and columns

The report appears to use a table named `Female_Completion` with fields such as:

- `Country`
- `Country Code`
- `Region`
- `Income Group`
- `Year`
- `Completion Rate`

In the cleaned CSV file, the column name is `Average of Completion Rate`. You may want to rename it to `Completion Rate` when rebuilding the model.

---

## 1) Average Completion Rate

```DAX
Average Completion Rate =
AVERAGE('Female_Completion'[Completion Rate])
```

Purpose:

- Returns the average completion rate across the current filter context.
- This is used in KPI cards and as a base for year-over-year comparisons.

---

## 2) Change Bar Colour

```DAX
Change Bar Colour =
IF(
    [Matched Country Change (pp)] >= 0,
    "#138B75",
    "#D36A57"
)
```

Purpose:

- Encodes positive country changes in green and negative changes in red.
- Used for conditional formatting in ranking or bar visuals.

---

## 3) Matched Country Count

```DAX
Matched Country Count =
VAR Countries =
    CALCULATETABLE(
        VALUES('Female_Completion'[Country Code]),
        REMOVEFILTERS('Female_Completion'[Year])
    )
RETURN
    COUNTROWS(
        FILTER(
            Countries,
            NOT ISBLANK(
                CALCULATE(
                    [Average Completion Rate],
                    REMOVEFILTERS('Female_Completion'[Year]),
                    'Female_Completion'[Year] = 2020
                )
            )
            &&
            NOT ISBLANK(
                CALCULATE(
                    [Average Completion Rate],
                    REMOVEFILTERS('Female_Completion'[Year]),
                    'Female_Completion'[Year] = 2024
                )
            )
        )
    )
```

Purpose:

- Counts how many countries have valid data for both 2020 and 2024.
- This ensures the comparison is calculated only for matched countries.

---

## 4) Change Rank

```DAX
Change Rank =
VAR ImprovementRank =
    RANKX(
        ALLSELECTED('Female_Completion'[Country]),
        [Matched Country Change (pp)],
        ,
        DESC,
        DENSE
    )
VAR DeclineRank =
    RANKX(
        ALLSELECTED('Female_Completion'[Country]),
        [Matched Country Change (pp)],
        ,
        ASC,
        DENSE
    )
RETURN
    IF(
        NOT ISBLANK([Matched Country Change (pp)])
            && (ImprovementRank <= 5 || DeclineRank <= 5),
        1,
        0
    )
```

Purpose:

- Flags the top five countries with the largest positive and negative changes.
- Useful for spotlighting the most notable movement in the dashboard.

---

## 5) Matched Country Change (pp)

```DAX
Matched Country Change (pp) =
VAR CountryList =
    CALCULATETABLE(
        VALUES('Female_Completion'[Country Code]),
        REMOVEFILTERS('Female_Completion'[Year])
    )
VAR CountryRates =
    ADDCOLUMNS(
        CountryList,
        "Rate2020",
            CALCULATE(
                [Average Completion Rate],
                REMOVEFILTERS('Female_Completion'[Year]),
                'Female_Completion'[Year] = 2020
            ),
        "Rate2024",
            CALCULATE(
                [Average Completion Rate],
                REMOVEFILTERS('Female_Completion'[Year]),
                'Female_Completion'[Year] = 2024
            )
    )
VAR MatchedCountries =
    FILTER(
        CountryRates,
        NOT ISBLANK([Rate2020])
            && NOT ISBLANK([Rate2024])
    )
RETURN
    AVERAGEX(
        MatchedCountries,
        [Rate2024] - [Rate2020]
    )
```

Purpose:

- Builds a list of countries with data for both 2020 and 2024.
- Computes the change for each matched country.
- Returns the average of those country-level changes.

Interpretation:

- This is not the same as “difference between the average 2024 value and the average 2020 value” across all countries.
- It is the average of matched-country changes, which can be a better measure when country coverage differs by year.

---

## 6) Reporting Countries

```DAX
Reporting Countries =
DISTINCTCOUNT('Female_Completion'[Country Code])
```

Purpose:

- Counts the number of country entries in the current filtered context.
- Useful for showing how many countries have data in a year or a selected subset.

---

## Common rebuild notes

If you rebuild this model from the CSV file, the main adjustment is to make sure the table is named exactly `Female_Completion` and that the completion-rate field is named `Completion Rate`.

A simple alternative is to rename the raw CSV column to `Completion Rate` and then use:

```DAX
Average Completion Rate =
AVERAGE('Female_Completion'[Completion Rate])
```

This keeps the measure names consistent with the report logic.

---

## Suggested documentation wording

The change measure is best explained as:

> `Matched Country Change (pp)` compares the completion rate for the same set of countries in 2020 and 2024, then averages the country-level percentage-point change across the matched set.

This is clearer and easier to defend than a generic statement that a region or group “improved by X%” when the data is in percentage points.

---

## Caveat on interpretation

Because some completion rates exceed 100%, it is important to treat the metric as a reported educational indicator rather than a literal percentage of a possible maximum. When presenting the story:

- focus on direction and magnitude of change
- use percentage points instead of percent growth unless the calculation is explicitly defined
- flag extraordinary values as possible reporting anomalies or data quality issues

This keeps the analysis transparent and avoids overstating what the data proves.

