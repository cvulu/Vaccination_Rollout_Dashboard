# DAX Measures

This document contains the core DAX logic used in the Vaccination Equity Dashboard.
Measures support KPI calculations, Māori vs non-Māori equity analysis, dynamic labelling,
and imputation logic to handle age categories.

Note: All measures following the KPI X naming pattern share identical logic — only the KPI number changes.
For clarity, only one full example is shown per pattern.

> All data included in this portfolio is fully anonymised.

### _KPI Targets | Overview page
Purpose: Retrieves the expected target value for a specific KPI number.

_KPI X Target = 
CALCULATE(
    SUM(KPI_combined[Expected]),
    FILTER(
        ALLSELECTED(KPI_combined),
        KPI_combined[KPI_#] = X
    )
)

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### _Heatmap KPI X % Achieved (Display) | Demographics page
Purpose: Returns the KPI success rate as a formatted percentage for display visuals.

_Heatmap KPI X % Achieved (Display) = 
FORMAT([_Heatmap KPI X % Achieved (Numeric)], "0%")

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### _Heatmap KPI X % Achieved (Numeric) | Demographics page
Purpose: Calculates the KPI success ratio (Actual ÷ Target) at the Collective level, while preventing incorrect aggregation when multiple collectives are visible.

_Heatmap KPI X % Achieved (Numeric) = 
VAR RowCollective = SELECTEDVALUE(Final_INPUT_table[Collective])
RETURN
IF(
    NOT ISINSCOPE(Final_INPUT_table[Collective]),
    BLANK(),
    IF(
        NOT CONTAINS(
            ALLSELECTED(Final_INPUT_table[Collective]),
            Final_INPUT_table[Collective],
            RowCollective
        ),
        BLANK(),
        DIVIDE(
            CALCULATE(
                [_KPI X Actual],
                REMOVEFILTERS(Final_INPUT_table[Collective]),
                KEEPFILTERS(Final_INPUT_table[Collective] = RowCollective)
            ),
            CALCULATE(
                [_KPI X Target],
                REMOVEFILTERS(Final_INPUT_table[Collective]),
                KEEPFILTERS(Final_INPUT_table[Collective] = RowCollective)
            ),
            0
        )
    )
)

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### _KPI 1 Actual
Purpose: Returns the total vaccination count for KPI 1 (all records).

_KPI 1 Actual = 
IF(ISBLANK(CALCULATE(
            SUM(Final_INPUT_table[Vaccination Count]),
            ALLSELECTED(Final_INPUT_table)
                    )
            ),
            0,
    CALCULATE(
        SUM(Final_INPUT_table[Vaccination Count]),
        ALLSELECTED(Final_INPUT_table)
            )
    )

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### _KPI 2 Actual
Purpose: Returns the total count of HPV vaccinations.

_KPI 2 Actual = 
IF(ISBLANK(CALCULATE(
    SUM(Final_INPUT_table[Vaccination Count]), 
    FILTER(
        ALLSELECTED(Final_INPUT_table),
        Final_INPUT_table[Vaccination Type_CLEAN] = "HPV"
            )
                    )
            ),
            0,
        CALCULATE(
            SUM(Final_INPUT_table[Vaccination Count]), 
                FILTER(
                ALLSELECTED(Final_INPUT_table),
                Final_INPUT_table[Vaccination Type_CLEAN] = "HPV"
                    )
                )
        )

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### _KPI 3 Actual
Purpose: Returns the total count of vaccinations administered to children aged 0–12 years.

_KPI 3 Actual = 
VAR ActualValue = 
    CALCULATE(
        SUM(Final_INPUT_table[Vaccination Count]),
        Final_INPUT_table[Age Group] IN { "Pēpi (0-4)", "Tamariki (5-12)" },
        ALLSELECTED(Final_INPUT_table)
    )
RETURN
    IF(ISBLANK(ActualValue), 0, ActualValue)

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### _KPI 4 Actual
Purpose: Returns the total count of non-HPV vaccinations administered to people aged 13 and older.

_KPI 4 Actual = 
VAR ActualValue = 
    CALCULATE(
        SUM(Final_INPUT_table[Vaccination Count]),
        Final_INPUT_table[Age Group] IN { "Rangatahi (13-17)", "Pākeke (18-65)", "Kaumatua (65+)" },
        Final_INPUT_table[Vaccination Type_CLEAN] <> "HPV",
        ALLSELECTED(Final_INPUT_table)
    )
RETURN
    IF(ISBLANK(ActualValue), 0, ActualValue)

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### _KPI X Colour | Overview page
Purpose: Defines colour logic for KPI cards or visuals (green = met/exceeded, red = shortfall).

_KPI X Colour = 
VAR Actual = Final_INPUT_table[_KPI X Actual]
VAR Target = KPI_combined[_KPI X Target]
VAR Gap = Actual - Target
RETURN
    SWITCH(
        TRUE(),
        Gap > 0, "#00A82E",     // green = exceeded
        Gap = 0, "#00A82E",     // green = met
        TRUE, "#D64554"         // red = shortfall or missing actual
    )

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### _KPI X Gap | Overview page
Purpose: Displays a text summary of whether the KPI was met, exceeded, or fell short — including the difference from target.

_KPI X Gap = 
VAR Actual = Final_INPUT_table[_KPI X Actual]
VAR Target = KPI_combined[_KPI X Target]
VAR Gap = Actual - Target
RETURN
    IF(
        Gap > 0,
        "Exceeded: " & UNICHAR(10) & UNICHAR(10003) & " " & FORMAT(Gap, "#,##0"),
            IF(Gap = 0,
            "Met." & UNICHAR(10) & UNICHAR(10003) & " " & "0",
                "Shortfall: " & UNICHAR(10) & UNICHAR(10060) & " " & FORMAT(ABS(Gap), "#,##0")
                )
            )

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### _Total Māori Percentage
Purpose: Calculates the proportion of vaccinations received by Māori clients.


_Total Māori Percentage = 
DIVIDE(
    SUM(Final_INPUT_table[Total Māori_CLEANED]),
    SUM(Final_INPUT_table[Vaccination Count]))

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### _Total Other Ethnicity
Purpose: Returns the total count of non-Māori vaccinations.

_Total Other Ethnicity = 
CALCULATE(SUM(Final_INPUT_table[Vaccination Count]), REMOVEFILTERS(Final_INPUT_table[Total Māori_CLEANED]))
-
CALCULATE(SUM(Final_INPUT_table[Total Māori_CLEANED]))

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### _Visualisations Dynamic Indicator
Purpose: Dynamically generates a descriptive title that reflects which region and year filters are active in the report.

_Visualisations Dynamic Indicator = 
VAR HasRegionFilter =
    COUNTROWS(VALUES(Final_INPUT_table[Region])) <>
    COUNTROWS(ALL(Final_INPUT_table[Region]))

VAR SelectedRegions =
    IF(
        HasRegionFilter,
        CONCATENATEX(
            VALUES(Final_INPUT_table[Region]),
            Final_INPUT_table[Region],
            ", "
        )
    )

VAR SelectedYears =
    CONCATENATEX(
        VALUES(Bridge_combined[Year]),
        Bridge_combined[Year],
        ", "
    )

RETURN
    IF(
        HasRegionFilter,
        "Showing Visualisations for Region(s): " & SelectedRegions &
            IF(NOT ISBLANK(SelectedYears), " (" & SelectedYears & ")", ""),
        "Showing Visualisations for: All Regions" &
            IF(NOT ISBLANK(SelectedYears), " (" & SelectedYears & ")", "")
    )
