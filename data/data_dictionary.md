Data Dictionary — Vaccination Rollout Dashboard

Purpose: Describe the structure of data used by the Power BI model for vaccination KPI and equity reporting (Māori vs Non-Māori).
Notes: Real programme data was used during development but is fully anonymised in this portfolio. No PHI/PII is published.

1) Source Tables
1.1 INPUT_table
Aggregated vaccination records by provider/collective/region and age bands.

Column	                        Type        Example	                        Description
Partner_submitted	            text	    "Yes"	                        Category used by other documents; dropped in final model.
Partner Name    	            text	    "ABC Health"	                Original provider name (removed in final model).
Vaccination Type	            text	    "Influenza"	                    Original vaccine type (removed in final model).
Vaccination Count	            whole	    22778	                        Total vaccinations (record total before age split).
Pēpi (0-4)      	            whole	    30871	                        Count for 0–4 age band (may contain nulls).
Tamariki (5-12) 	            whole	    14576	                        Count 5–12.
Rangatahi (13-17)	            whole	    3947	                        Count 13–17.
Pākeke (18-65)  	            whole	    12511	                        Count 18–65.
Kaumatua (65+)               	whole	    2347	                        Count 65+.
% Māori         	            decimal	    0.54	                        Proportion Māori (0–1) for the record.
Count Check (auto-calculates)	whole	    –	                            Authoring artefact; not used in final model.
Total Māori (auto-calculates)	whole	    –	                            Authoring artefact; not used in final model.
Partner_CLEAN	                text	    "Partner Clean 23"	            Anonymised provider label.
Collective_CLEAN	            text	    "Collective 12"	                Anonymised collective label.
Region_CLEAN	                text	    "Te Tai Hauāuru"	            Region label.
Vaccination Type_CLEAN	        text	    "Influenza"	                    Standardised vaccine type.
Key	                            text	    `"Collective12::FY24-25"`       Primary key.
Quarter	                        text	    "Q1"	                        Financial quarter.
Year	                        text	    "FY24–25"	                    Financial year (text).
Comments	                    text	    –               	            Free text; dropped in final model.


1.2 KPI_combined
Targets per collective/KPI/year.

Column	                Type	        Example	                        Description
Collective	            text	        "Collective 12"	                Collective name (anonymised).
KPI_#	                whole	        1	                            KPI number (1–4).
Expected	            whole	        3189	                        Target value for the KPI.
Year	                text	        "FY24–25"	                    Financial year.
Key	                    text	        `"Collective12	FY24-25"`       Primary key.

KPI definitions:
- KPI 1: Total vaccinations (all)
- KPI 2: HPV vaccinations
- KPI 3: Child vaccinations (0–12)
- KPI 4: Other (13+ and non-HPV)


1.3 Bridge_combined
Simple bridge for Year/Quarter slicers.

Column	Type	Example	Description
Year	text	"FY24–25"	Financial year.
Quarter	text	"Q1"	Financial quarter.


1.4 Address_table
Geocoding for map visual.

Column  	    Type              	Example	                        Description
Partner 	    text	            "Partner Clean 23"	            Anonymised provider label.
Address 	    text	            "123 Example Rd, Auckland"	    Generalised address.
Latitude    	decimal	            -36.8509	                    Latitude.
Longitude   	decimal	            174.7645	                    Longitude.
Region      	text	            "Tāmaki Makaurau"	            Region.


2) Transformed Tables (Power Query Output)
2.1 AgeGroupVisualTable (long format)
Unpivots age columns; marks origin as “Reported”.

Column          	            Type	            Example                     Description
Partner_CLEAN	                text	            "Partner Clean 23"	        Provider (anonymised).
Collective_CLEAN	            text	            "Collective 12"	            Grouping.
Region_CLEAN	                text	            "Te Tai Tokerau"	        Region.
Vaccination Type_CLEAN	        text	            "Influenza"	                Standardised vaccine type.
Key	                            text	            `"Collective12	FY24-25"`   Primary key.
Quarter	                        text	            "Q1"	                    Quarter.
Year	                        text	            "FY24–25"	                Year.
Age Group	                    text	            "Pēpi (0-4)"	            Age band (domain below).
Age Group Count	                whole	            1567	                    Count for the age band.
RecordedSource	                text	            "Reported"	                Row provenance.
% Māori	                        decimal         	0.63	                    Māori proportion for the row.


2.2 MissingAgeGroupRows (imputed)
Creates “Not Specified” rows where age-band totals < overall.

Column	                        Type	    Example	            Description
Partner_CLEAN	                text	    –	                From source.
Collective_CLEAN	            text	    –	                From source.
Region_CLEAN	                text	    –	                From source.
Vaccination Type_CLEAN	        text	    –	                From source.
Key	                            text	    –	                From source.
Quarter	                        text	    –	                From source.
Year	                        text	    –	                From source.
Age Group	                    text	    "Not Specified"	    Imputed bucket.
Age Group Count	                whole	    89	                Missing portion.
RecordedSource	                text	    "Imputed"	        Row provenance.
% Māori	                        decimal	    0.46	            Proportion applied to imputed count.


2.3 Final_INPUT_Table (fact for model)
Append of reported + imputed; renamed and rounded for visuals.

Column             	        Type	    Example	                        Description
Partner	                    text	    "Partner Clean 23"	            Provider (final name).
Collective	                text	    "Collective 12"	                Collective (final name).
Region	                    text	    "Te Tai Hauāuru"	            Region.
Vaccination Type_CLEAN	    text	    "Influenza"	                    Vaccine.
Key	                        text	    `"Collective12	FY24-25"`       Primary key.
Quarter	                    text	    "Q1"	                        Period.
Year	                    text	    "FY24–25"	                    Period.
Age Group	                text	    "Pēpi (0-4)"	                Age band.
Vaccination Count	        whole	    480	                            Count used in visuals.
RecordedSource	            text	    "Reported"/"Imputed"	        Provenance.
% Māori	                    decimal	    0.70	                        Māori proportion.
Total Māori_CLEANED	        whole	    336	                            Rounded Māori count (Number.RoundUp of % Māori × Vaccination Count).
Total Māori	                decimal	    pre-rename intermediate	        Kept for lineage (may not be exposed).


3) Value Domains
Age Group
- Pēpi (0-4)
- Tamariki (5-12)
- Rangatahi (13-17)
- Pākeke (18-65)
- Kaumatua (65+)
- Not Specified (imputed)

Vaccination Type (standardised examples)
- Influenza
- MenB
- COVID-19
- PCV
- DTaP-IPV-HepB/Hib
- MMR
- Other
- RV
- Tdap
- VV
- HPV
- Hib
- IPV
- ZV
- HepB

RecordedSource
- Reported — supplied in source
- Imputed — created to reconcile missing age allocations


4) KPI Definitions (semantic layer)
KPI	        Definition	                          Filter logic
KPI 1	    Total vaccinations (all)	          Sum of Vaccination Count
KPI 2	    HPV vaccinations	                  Vaccination Type_CLEAN = "HPV"
KPI 3	    Child vaccinations (0–12)	          Age Group IN {"Pēpi (0-4)","Tamariki (5-12)"}
KPI 4	    Other vaccinations (13+ non-HPV)	  Age Group IN {"Rangatahi (13-17)","Pākeke (18-65)","Kaumatua (65+)"} AND Vaccination Type_CLEAN <> "HPV"

Targets are supplied by KPI_combined[Expected] per Collective/Year.


5) Relationships (high level)
- Final_INPUT_Table[Key] ↔ KPI_combined[Key] (many-to-one)
- Final_INPUT_Table[Year] ↔ Bridge_combined[Year] (many-to-one)
- Final_INPUT_Table[Partner] ↔ Address_table[Partner] (optional, for maps)

Notes:
- Direction: single (from dimension/lookup to fact).
- Region and Collective are treated as attributes in Final_INPUT_Table.
- No Row-Level Security published in this portfolio version.


6) Data Quality & Transform Notes
- Null handling: age bands with nulls are treated as 0 prior to arithmetic.
- Imputation rule: if Σ(age bands) < Vaccination Count, allocate the difference to Not Specified.
- Māori totals: computed at long-row level and rounded up for reporting (Number.RoundUp).
- Anonymisation: Provider and Collective labels are canonicalised to “Clean” names.


7) Reproducibility (without sensitive data)
To rebuild the model with synthetic data:
1. Create CSV/XLSX tables matching the column sets above.
2. Load into Power BI and apply the scripts in /transformations/ (Power Query M and DAX).
3. Replace map coordinates with public or jittered points.