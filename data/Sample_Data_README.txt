Purpose: This document provides context for the data used in this Power BI project.

Overview
This dashboard was built using real programme data that has been fully anonymised and de-identified to protect organisations, collectives, and individuals.
No personally identifiable information or operational identifiers remain.

Data Access
The original data is not included in this public repository for privacy and compliance reasons.
All published exports (e.g., PDFs, screenshots) are safe for public viewing.

Data Structure (Summary)
The source data consisted of multiple linked tables:
- INPUT_table — aggregated vaccination counts by age group and ethnicity.
- KPI_combined — yearly KPI targets by collective and region.
- Bridge_combined — year and quarter bridge for time-based filtering.
- Address_table — anonymised location coordinates for Power BI map visuals.
A full schema reference is available in data_dictionary.md.

Reproducibility

To reproduce the project structure without sensitive data:
1. Create placeholder tables matching the same column structure.
2. Populate with synthetic or publicly available values.
3. Load into Power BI and apply the transformation scripts in /transformations/.