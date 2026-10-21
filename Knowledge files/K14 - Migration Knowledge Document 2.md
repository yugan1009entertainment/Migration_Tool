
The user is uploading multiple files to be used as context for further actions and has requested to save them without generating responses until prompted.
One file, "attribute_summary_updated_sheet1.txt", contains the schema for the table "Attribute Summary Updated_Sheet1" with columns such as Dataset Vendor, Therapeutic Area, Indication, Refresh Date, Attribute Category, Elements, # Patients, Cohort considered, % Patients, and calculated fields like % Patient - Final and # Patients - Numeric.
The file includes column data types, lineage tags, display order, summarization settings, formulas for calculated columns, and information about the data import source (an Excel file from a SharePoint link).The issue is that rendered Markdown tables lose their formatting when copied directly from the chat UI. The fix is to wrap all tables inside code blocks so you can copy the raw Markdown text. Below is the full Knowledge Base rewritten with every table inside a code block.


K14 - Tableau to Power BI Migration Knowledge Base
Project: 24363_DRWDS_Oncology_Capability Assessment
Generated: 2026-06-12 | Classification: Internal BMS Use Only

1. PROJECT OVERVIEW
| Attribute                  | Detail                                                              |
|----------------------------|---------------------------------------------------------------------|
| Workbook Name              | 24363-DRWDS_Oncology_CapabilityAssessment                           |
| Tableau Source Build       | 2023.1.4 (20231.23.0711.1149)                                       |
| Tableau Server             | https://tableau.bms.com                                             |
| Site                       | PRODUCTION                                                          |
| Revision                   | 1.0                                                                 |
| Power BI Model Name        | 24363_DRWDS_Oncology_Capability Assessment- model                   |
| PBI Compatibility Level    | 1550                                                                |
| PBI Culture                | en-US                                                               |
| Domain                     | BMS Real World Data Science (RWDS) - Oncology                       |
| Vendor                     | HealthVerity                                                        |
| Migration Year             | 2026                                                                |
| Data Models Used           | Data Model 1-10, 14 (Excel files on BMS SharePoint)                 |


2. SOURCE AND TARGET ARCHITECTURE
2.1 Tableau Architecture
Tableau Workbook (.twb)
├── Parameters Datasource (inline)
│   ├── Parameter 1: Yearly/Monthly
│   ├── Parameter 2: start date/end date
│   └── Parameter 3: Dashboard Refresh Date
├── Datasource: Outlier Patient Summary (Excel → Hyper Extract)
├── Datasource: Header Summary (Excel → Hyper Extract)
├── Datasource: patient_cohort (Excel → Hyper Extract)
├── Datasource: Patient Overall (Excel → Hyper Extract)
├── Datasource: Executive Summary (Excel → Hyper Extract)
├── Datasource: Use Case Analysis Updated (Excel → Hyper Extract)
└── Additional datasources for Cost, Attribute, Data Quality, Screen Config

2.2 Power BI Architecture
Power BI PBIP Project
├── model.txt (Tabular model, 1550 compat level)
├── Tables
│   ├── Measures Table (dedicated measures-only table)
│   ├── Yearly Monthly (Parameter 1)
│   ├── start date end date (Parameter 2)
│   ├── Dashboard Refresh Date (Parameter 3)
│   ├── Outlier Patient Summary_5 Point Summary
│   ├── Header Summary_Sheet1
│   ├── patient_cohort_Sheet1
│   ├── Patient Overall_Sheet1
│   ├── Executive Summary_Sheet1
│   ├── Use Case Analysis Updated_Sheet1
│   ├── Attribute Summary Updated_Sheet1
│   ├── Use Case Analysis - Examples_Sheet1
│   ├── Cost Analysis_Sheet1
│   ├── Data Quality_Sheet1
│   ├── Screen Config_Sheet1
│   ├── Table (lookup: Attribute Name sort order)
│   └── Table (2) (lookup: Category Type sort key)
├── relationships.txt
├── expressions.txt (Power Query parameters)
├── DateTableTemplate (auto-generated)
└── LocalDateTable (auto-generated)


3. DATA SOURCE MAPPING
3.1 Tableau to Power BI Table Mapping
| #  | Tableau Datasource Name         | Tableau Source File                  | Power BI Table Name                        | PBI Source Excel    | Sheet          |
|----|----------------------------------|--------------------------------------|--------------------------------------------|---------------------|----------------|
| 1  | Outlier Patient Summary          | Model 8 - outliers.xlsx              | Outlier Patient Summary_5 Point Summary    | Data Model 8.xlsx   | 5 Point Summary|
| 2  | Header Summary                   | Model 14- header summary2.xlsx       | Header Summary_Sheet1                      | Data Model 14.xlsx  | Sheet1         |
| 3  | patient_cohort                   | Model 2- patient cohort.xlsx         | patient_cohort_Sheet1                      | Data Model 2.xlsx   | Sheet1         |
| 4  | Patient Overall                  | Model 1- patient overall.xlsx        | Patient Overall_Sheet1                     | Data Model 1.xlsx   | Sheet1         |
| 5  | Executive Summary                | Model 4- executive summary.xlsx      | Executive Summary_Sheet1                   | Data Model 4.xlsx   | Sheet1         |
| 6  | Use Case Analysis Updated        | Model 7- Use case analysis.xlsx      | Use Case Analysis Updated_Sheet1           | Data Model 7.xlsx   | Sheet1         |
| 7  | Attribute Summary                | Model 3.xlsx (inferred)              | Attribute Summary Updated_Sheet1           | Data Model 3.xlsx   | Sheet1         |
| 8  | Cost Analysis                    | Model 10.xlsx                        | Cost Analysis_Sheet1                       | Data Model 10.xlsx  | Sheet1         |
| 9  | Data Quality                     | Model 5.xlsx                         | Data Quality_Sheet1                        | Data Model 5.xlsx   | Sheet1         |
| 10 | Screen Config                    | Model 10.xlsx                        | Screen Config_Sheet1                       | Data Model 10.xlsx  | Sheet1         |

3.2 Connection Type Change
| Attribute        | Tableau                              | Power BI                                          |
|------------------|--------------------------------------|---------------------------------------------------|
| Connection Type  | Excel Direct + Hyper Extract         | Excel via Web.Contents() (SharePoint Online)      |
| Refresh Mode     | Tableau Extract (.hyper files)       | Power BI Import Mode                              |
| Query Pattern    | N/A                                  | Table.PromoteHeaders() applied in all M queries   |


4. PARAMETERS MAPPING
4.1 Tableau to Power BI Parameter Mapping
| Tableau Parameter | Tableau Caption       | Type          | Default       | Power BI Equivalent          | PBI Type     | PBI Expression GUID                          |
|-------------------|-----------------------|---------------|---------------|------------------------------|--------------|----------------------------------------------|
| Parameter 1       | Yearly/Monthly        | String (list) | "1"           | Yearly Monthly table         | Text (list)  | b2a7c08e-7fe5-4ace-bab8-a4410a38192e         |
| Parameter 2       | start date/end date   | String (list) | "1"           | start date end date table    | Text (list)  | 8ca5330a-e08a-4661-9566-993a94ec9e5b         |
| Parameter 3       | Dashboard Refresh Date| Date          | #2023-09-06#  | Dashboard Refresh Date table | Date         | 71b5b1bc-4667-4551-af29-b56663543afe         |

4.2 Parameter Implementation Notes
| Tableau Approach                                              | Power BI Approach                                                          |
|---------------------------------------------------------------|----------------------------------------------------------------------------|
| Parameters defined inline in the .twb datasource block        | Parameters implemented as single-row lookup tables                         |
| Parameter values referenced via [Parameter N] in calculations | Referenced via SELECTEDVALUE() or slicer binding                           |
| Parameter 3 (date) = #2023-09-06#                             | Stored as #date(2023,09,06) in M; linked to LocalDateTable via rel 7aa34c2c|
| Parameter 1 alias: Yearly                                     | PBI: value "1" in ParameterTableKeyColumn                                  |
| Parameter 2 alias: Event Start Date                           | PBI: value "1" in ParameterTableKeyColumn                                  |


5. CALCULATED FIELDS AND MEASURES MAPPING
5.1 Outlier Patient Summary - Calculations
| Tableau Calculated Field               | Tableau Formula                                                                                         | Power BI Name                                                        | Power BI DAX                                                                                                                                 | Type               |
|----------------------------------------|---------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------|--------------------|
| % of Patient Outlier                   | SUM(if [Patient Type]='Outlier' then [# of Outlier Patients] END) / MAX([# of Patient Captured])        | % of Patient Outlier                                                 | DIVIDE(CALCULATE(SUMX(...IFERROR(VALUE([# of Outlier Patients]),0)),FILTER(...[Patient Type]="Outlier")),MAXX(...IFERROR(VALUE([# of Patient Captured]),0))) | Measure |
| % of Patient Captured                  | SUM([# of Patient Captured]) / MAX([Total Patient Count])                                               | % of Patient Captured                                                | DIVIDE(SUMX(...IFERROR(VALUE([# of Patient Captured]),0)),MAXX(...IFERROR(VALUE([Total Patient Count]),0)))                                   | Measure            |
| PctTotal % of Patient Captured         | Table Calc: PctTotal                                                                                    | PctTotal_% of Patient Captured_Outlier Patient Summary_5 Point Summary | Placeholder - table calc converted to DAX DIVIDE context                                                                                   | Measure            |
| Category and Units                     | [Category_type] + ' (in ' + [Units] + ')'                                                               | Category and Units                                                   | [Category_type] & " (in " & [Units] & ")"                                                                                                    | Calculated Column  |
| sort (new in PBI)                      | No Tableau equivalent                                                                                   | sort                                                                 | RELATED('Table (2)'[s])                                                                                                                      | Calculated Column  |

5.2 Patient Overall - Calculations
| Tableau Calculated Field           | Tableau Formula                                                                                                | Power BI Name               | Power BI DAX                                                                                                                                                                      | Type              |
|------------------------------------|----------------------------------------------------------------------------------------------------------------|-----------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------|
| % of Patients                      | SUM([Number of Patients]) / MAX([Total Patients])                                                              | % of Patients (PO)          | DIVIDE(SUMX(...IFERROR(VALUE([Number of Patients]),0)),MAXX(...IFERROR(VALUE([Total Patients]),0)))                                                                                | Measure           |
| Top 10 MEds                        | [Rank]<=10 OR SUM([BMS Specific Medication flag])=1                                                            | Top 10 MEds                 | IF(OR([Rank]<=10,SUM([BMS Specific Medication flag])=1),TRUE(),FALSE())                                                                                                            | Measure           |
| Rank                               | RANK(SUM({FIXED [Category Type],[BMS Specific Medication flag]:SUM([Number of Patients])}),'desc')             | Rank                        | RANKX(ALL([Category Type],[BMS Specific Medication flag]),CALCULATE(SUM([Number of Patients]),REMOVEFILTERS(...),KEEPFILTERS(VALUES([Category Type])),KEEPFILTERS(VALUES([BMS Specific Medication flag]))),, DESC, DENSE) | Measure |
| Sum of Number of Patients (new PBI)| -                                                                                                              | Sum of Patients (PO)        | SUMX(...,IFERROR(VALUE([Number of Patients]),0))                                                                                                                                   | Measure           |
| Total Patients Headline (new PBI)  | -                                                                                                              | Total Patients (PO)         | MAXX(...,IFERROR(VALUE([Total Patients]),0))                                                                                                                                       | Measure           |
| Sum of Patients Display (new PBI)  | -                                                                                                              | Sum of Patients (PO) display| "(N=" & FORMAT(_PatientCount,"#,##0") & ")"                                                                                                                                       | Measure           |
| % Patients Bar Color (new PBI)     | -                                                                                                              | % Patients Bar Color        | SWITCH(TRUE(),ISBLANK(_Value),"#EDEDED",_Value<0.30,"#E6E6E6",_Value<0.40,"#D8D2DD",_Value<0.50,"#C4B2CF",_Value<0.60,"#9F7EAD",_Value<0.75,"#7E5590","#5B2D75")                   | Measure           |

5.3 Patient Cohort - Calculations
| Tableau Calculated Field       | Tableau Formula                                               | Power BI Name           | Power BI DAX                                                                                                      | Type                       |
|--------------------------------|---------------------------------------------------------------|-------------------------|-------------------------------------------------------------------------------------------------------------------|----------------------------|
| % of Patients                  | SUM([Number of Patients]) / MAX([Total Patients])             | % of Patients           | DIVIDE(SUMX(...IFERROR(VALUE([Number of Patients]),0)),MAXX(...IFERROR(VALUE([Total Patients]),0)))                | Measure                    |
| States                         | if [Main Category]='States' THEN [Category Type] END          | States                  | IF([Main Category]="States",[Category Type],BLANK())                                                              | Calculated Column          |
| Category Type Sort (new PBI)   | -                                                             | Category Type Sort      | IFERROR(VALUE([Category Type]),9999)                                                                              | Calculated Column (hidden) |
| Country (new PBI)              | -                                                             | Country                 | "United States"                                                                                                   | Calculated Column (hidden) |
| State Geo (new PBI)            | -                                                             | State Geo               | SWITCH([Category Type],"PR","Puerto Rico, United States","DC","District of Columbia, United States",[Category Type] & ", United States") | Calculated Column (hidden) |
| State Lat (new PBI)            | -                                                             | State Lat               | Full SWITCH with lat coords for all 50 states + PR                                                                | Calculated Column (hidden) |
| State Lng (new PBI)            | -                                                             | State Lng               | Full SWITCH with lng coords for all 50 states + PR                                                                | Calculated Column (hidden) |

5.4 Cost Analysis - Calculations
| Tableau Calculated Field | Tableau Formula                                                                                       | Power BI Name      | Power BI DAX                                                                           | Type              |
|--------------------------|-------------------------------------------------------------------------------------------------------|--------------------|----------------------------------------------------------------------------------------|-------------------|
| Condition Cost           | if [Main Category]='Cost' THEN 1 else 0 end                                                           | Condition Cost     | IF([Main Category]="Cost",1,0)                                                         | Calculated Column |
| Show/Hide Cost           | if SUM([Condition Cost])>=1 THEN "" else "The data in the view is not available..." END                | Show/Hide Cost     | IF(SUM([Condition Cost])>=1,"","The data in the view is not available for the selected indication") | Measure |
| Non-Outlier Range        | Referenced Min/Max columns                                                                            | Non-Outlier Range  | "Not Available" (PLACEHOLDER - columns missing from Cost model)                        | Calculated Column |

5.5 Data Quality - Calculations
| Tableau Calculated Field           | Tableau Formula                                               | Power BI Name                  | Power BI DAX                                                                                                   | Type              |
|------------------------------------|---------------------------------------------------------------|--------------------------------|----------------------------------------------------------------------------------------------------------------|-------------------|
| Data Quality Issue                 | [DQ Issue Name] + ' :' + [Details]                            | Data Quality Issue             | [DQ Issue Name] & " :" & [Details]                                                                             | Calculated Column |
| % of Patients                      | SUM([Number of Patients]) / MAX([Total patients])             | % of Patients 1                | DIVIDE(SUMX(...IFERROR(VALUE([Number of Patients]),0)),MAXX(...IFERROR(VALUE([Total patients]),0)))             | Measure           |
| Number of Patients Display (new PBI)| -                                                            | Number of Patients Display     | IF(_Value="" OR UPPER(_Value)="NA" OR UPPER(_Value)="N/A","Null",FORMAT(IFERROR(VALUE(_CleanValue),0),"#,##0")) | Calculated Column |

5.6 Attribute Summary - Calculations
| Tableau Calculated Field    | Tableau Formula                    | Power BI Name        | Power BI DAX                                                                                                                   | Type              |
|-----------------------------|------------------------------------|----------------------|--------------------------------------------------------------------------------------------------------------------------------|-------------------|
| % Patient - Final           | Text conversion of % patients      | % Patient - Final    | IFERROR(VALUE([% Patients]),0)                                                                                                 | Calculated Column |
| % Patient - Final-dual      | Dual-axis variant                  | % Patient - Final-dual | IFERROR(VALUE([% Patients]),0)                                                                                               | Calculated Column |
| # Patients - Numeric        | Text to number conversion          | # Patients - Numeric | VAR _TextValue=TRIM([# Patients]&"") VAR _CleanValue=SUBSTITUTE(_TextValue,",","") RETURN IF(_TextValue=""||_TextValue="-",0,IFERROR(VALUE(_CleanValue),0)) | Calculated Column |

5.7 Use Case Analysis - Calculations (New in Power BI)
| Power BI Measure       | DAX Formula                                                                                                                  | Notes                                             |
|------------------------|------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------|
| Use Case Desc Concat   | CONCATENATEX(...,"▸ " & [Use Case Description],UNICHAR(10),[Use Case Description],ASC)                                       | No Tableau equivalent - new PBI enhancement       |
| Fulfillment Color      | SWITCH(TRUE(),SELECTEDVALUE([Fulfillement Index])="Can be fulfilled","#1a9850",SELECTEDVALUE([Fulfillement Index])="Can not be fulfilled","#d73027","#e67e22") | No Tableau equivalent - color coding |

5.8 Measures Table - Static and Navigation Labels
| Power BI Measure     | Value                                                                    | Tableau Equivalent         |
|----------------------|--------------------------------------------------------------------------|----------------------------|
| View NA              | "THE DATA IN THE VIEW IS NOT AVAILABLE FOR THE SELECTED INDICATION"      | Inline string              |
| 'Gender'             | "GENDER"                                                                 | Label                      |
| 'Race'               | "RACE"                                                                   | Label                      |
| 'Ethnicity'          | "ETHNICITY"                                                              | Label                      |
| 'Executive Summary'  | "EXECUTIVE SUMMARY"                                                      | Dashboard tab label        |
| 'Data Overview'      | "DATA OVERVIEW"                                                          | Dashboard tab label        |
| 'Attribute Summary'  | "ATTRIBUTE SUMMARY"                                                      | Dashboard tab label        |
| 'Cohort Assessment'  | "COHORT ASSESSMENT"                                                      | Dashboard tab label        |
| 'Cost Analysis'      | "COST ANALYSIS"                                                          | Dashboard tab label        |
| 'Use Case Analysis'  | "USE CASE ANALYSIS"                                                      | Dashboard tab label        |
| 'Next>'              | "NEXT>"                                                                  | Navigation button          |
| '<Prev'              | "<PREV"                                                                  | Navigation button          |
| INDEX()              | RANKX(ALL('Measures Table'),1)                                           | Tableau INDEX() table calc |
| '' / ''  1           | ""                                                                       | Empty string placeholders  |


6. COLUMNS MAPPING BY TABLE
6.1 Outlier Patient Summary (5 Point Summary sheet)
| #  | Tableau Column          | Tableau Type | PBI Column              | PBI DataType | Notes                                                                                  |
|----|-------------------------|--------------|-------------------------|--------------|----------------------------------------------------------------------------------------|
| 1  | Dataset Vendor          | string       | Dataset Vendor          | string       | Direct map                                                                             |
| 2  | Therapeutic Area        | string       | Therapeutic Area        | string       | Direct map                                                                             |
| 3  | Indication              | string       | Indication              | string       | Direct map                                                                             |
| 4  | Refresh Date            | string       | Refresh Date            | string       | Direct map                                                                             |
| 5  | Category                | string       | Category                | string       | Direct map                                                                             |
| 6  | Category_type           | string       | Category_type           | string       | Direct map; used in sort relationship                                                  |
| 7  | Patient Type            | string       | Patient Type            | string       | Direct map; filter value "Outlier" used in measure                                     |
| 8  | Units                   | string       | Units                   | string       | Direct map                                                                             |
| 9  | Total Patient Count     | integer      | Total Patient Count     | string       | Type changed - PBI stores as string, converts in DAX with IFERROR(VALUE())             |
| 10 | # of Patient Captured   | integer      | # of Patient Captured   | string       | Type changed                                                                           |
| 11 | # of Outlier Patients   | real         | # of Outlier Patients   | string       | Type changed                                                                           |

6.2 Header Summary (Sheet1)
| #  | Tableau Column   | PBI Column   | Type   | Notes      |
|----|------------------|--------------|--------|------------|
| 1  | Dataset Vendor   | Dataset Vendor   | string | Direct map |
| 2  | Therapeutic Area | Therapeutic Area | string | Direct map |
| 3  | Indication       | Indication       | string | Direct map |
| 4  | Refresh Date     | Refresh Date     | string | Direct map |
| 5  | Screen Name      | Screen Name      | string | Direct map |
| 6  | Header           | Header           | string | Direct map |
| 7  | Insights         | Insights         | string | Direct map |

6.3 patient_cohort (Sheet1)
| #  | Tableau Column              | Tableau Type                    | PBI Column                       | PBI Type           | Notes                                                                              |
|----|-----------------------------|---------------------------------|----------------------------------|--------------------|------------------------------------------------------------------------------------|
| 1  | Dataset Vendor              | string                          | Dataset Vendor                   | string             | Direct map                                                                         |
| 2  | Therapeutic Area            | string                          | Therapeutic Area                 | string             | Direct map                                                                         |
| 3  | Indication                  | string                          | Indication                       | string             | Direct map                                                                         |
| 4  | Refresh Date                | string (date-parse: yyyy/MM)    | Refresh Date                     | string             | Date parsing dropped in PBI                                                        |
| 5  | Sub Category                | string                          | Sub Category                     | string             | Direct map                                                                         |
| 6  | Main Category               | string                          | Main Category                    | string             | Direct map                                                                         |
| 7  | Total Patients              | integer                         | Total Patients                   | string             | Type changed                                                                       |
| 8  | Category Type               | string                          | Category Type                    | string             | Aliases 0.50→50% etc. in Tableau; handle via Power Query replace or calc column    |
| 9  | Number of Patients          | real                            | Number of Patients               | string             | Type changed                                                                       |
| -  | (new)                       | -                               | BMS Specific Medication flag     | string             | Added in PBI                                                                       |
| -  | (new)                       | -                               | Category Type Sort               | int64 (hidden)     | Numeric sort key                                                                   |
| -  | (new)                       | -                               | Country                          | string (hidden)    | Geocoding context                                                                  |
| -  | (new)                       | -                               | State Geo                        | string (hidden)    | State + country geocode                                                            |
| -  | (new)                       | -                               | State Lat                        | double (hidden)    | Latitude for map visual                                                            |
| -  | (new)                       | -                               | State Lng                        | double (hidden)    | Longitude for map visual                                                           |

6.4 Patient Overall (Sheet1)
| #  | Tableau Column                  | Tableau Type | PBI Column                       | PBI Type | Notes                                                                                         |
|----|---------------------------------|--------------|----------------------------------|----------|-----------------------------------------------------------------------------------------------|
| 1  | Dataset Vendor                  | string       | Dataset Vendor                   | string   | Direct map                                                                                    |
| 2  | Therapeutic Area                | string       | Therapeutic Area                 | string   | Direct map                                                                                    |
| 3  | Indication                      | string       | Indication                       | string   | Direct map                                                                                    |
| 4  | Refresh Date                    | string       | Refresh Date                     | string   | Direct map                                                                                    |
| 5  | Sub Category                    | string       | Sub Category                     | string   | Direct map                                                                                    |
| 6  | Main Category                   | string       | Main Category                    | string   | Direct map                                                                                    |
| 7  | Total Patients                  | integer      | Total Patients                   | string   | Type changed                                                                                  |
| 8  | Category Type                   | string       | Category Type                    | string   | Direct map                                                                                    |
| 9  | Number of Patients              | real         | Number of Patients               | string   | Type changed                                                                                  |
| 10 | BMS Specific Medication flag    | integer      | BMS Specific Medication flag     | string   | Aliases: 0=Other Brand Medication, 1=BMS Brand Medication in Tableau; handle via calc column  |

6.5 Executive Summary (Sheet1)
| #  | Tableau Column     | PBI Column              | Type   | Notes                                              |
|----|--------------------|-------------------------|--------|----------------------------------------------------|
| 1  | Dataset Vendor     | Dataset Vendor          | string | Direct map                                         |
| 2  | Therapeutic Area   | Therapeutic Area        | string | Direct map                                         |
| 3  | Indication         | Indication              | string | Direct map                                         |
| 4  | Refresh Date       | Refresh Date            | string | Direct map                                         |
| 5  | Main Category Name | Main Category Name      | string | Direct map                                         |
| 6  | Category Name      | Category Name           | string | Direct map                                         |
| 7  | Category Value     | Category Value          | string | Direct map                                         |
| -  | (new)              | Column8, Column9, Column10, Column11 | string | Extra columns from Excel; no Tableau equivalent |

6.6 Use Case Analysis Updated (Sheet1)
| #  | Tableau Column       | PBI Column           | Type   | Notes                                                                   |
|----|----------------------|----------------------|--------|-------------------------------------------------------------------------|
| 1  | Dataset Vendor       | Dataset Vendor       | string | Direct map                                                              |
| 2  | Therapeutic Area     | Therapeutic Area     | string | Direct map                                                              |
| 3  | Indication           | Indication           | string | Direct map                                                              |
| 4  | Refresh Date         | Refresh Date         | string | Direct map                                                              |
| 5  | Category             | Category             | string | Direct map                                                              |
| 6  | Main Category        | Main Category        | string | Direct map                                                              |
| 7  | Sub Category         | Sub Category         | string | Direct map                                                              |
| 8  | Use Case Category    | Use Case Category    | string | Direct map                                                              |
| 9  | Use Case Description | Use Case Description | string | Direct map                                                              |
| 10 | Fulfillement Index   | Fulfillement Index   | string | Direct map; typo "Fulfillement" preserved in both Tableau and Power BI  |

6.7 Attribute Summary Updated (Sheet1)
| #  | PBI Column         | Type   | Notes                                                      |
|----|--------------------|--------|------------------------------------------------------------|
| 1  | Dataset Vendor     | string | -                                                          |
| 2  | Therapeutic Area   | string | -                                                          |
| 3  | Indication         | string | -                                                          |
| 4  | Refresh Date       | string | -                                                          |
| 5  | Attribute Category | string | -                                                          |
| 6  | Elements           | string | -                                                          |
| 7  | # Patients         | string | Stored as text; converted via # Patients - Numeric column  |
| 8  | Cohort considered  | string | -                                                          |
| 9  | % Patients         | string | Stored as text; converted via % Patient - Final column     |
| -  | Column10, Column11 | string | Extra columns from Excel source                            |

6.8 Cost Analysis (Sheet1)
| #  | PBI Column                   | Type   | Notes                         |
|----|------------------------------|--------|-------------------------------|
| 1  | Dataset Vendor               | string | -                             |
| 2  | Therapeutic Area             | string | -                             |
| 3  | Indication                   | string | -                             |
| 4  | Refresh Date                 | string | SummarizationSetBy = Automatic|
| 5  | Main Category                | string | -                             |
| 6  | Category                     | string | -                             |
| 7  | Category_type                | string | -                             |
| 8  | BMS Specific Medication flag | string | -                             |
| 9  | # patients                   | string | -                             |
| 10 | Total patients               | string | -                             |

6.9 Data Quality (Sheet1)
| #  | PBI Column           | Type   | Notes |
|----|----------------------|--------|-------|
| 1  | Dataset Vendor       | string | -     |
| 2  | Therapeutic Area     | string | -     |
| 3  | Indication           | string | -     |
| 4  | Refresh Date         | string | -     |
| 5  | Main Category        | string | -     |
| 6  | Table Name           | string | -     |
| 7  | DQ Issue Name        | string | -     |
| 8  | Details              | string | -     |
| 9  | Total patients       | string | -     |
| 10 | Number of Patients   | string | -     |

6.10 Screen Config (Sheet1)
| #  | PBI Column                   | Type   | Notes                 |
|----|------------------------------|--------|-----------------------|
| 1  | Dataset Vendor               | string | Cross-filter anchor   |
| 2  | Therapeutic Area             | string | -                     |
| 3  | Indication                   | string | -                     |
| 4  | Refresh Date                 | string | -                     |
| 5  | Main Category                | string | -                     |
| 6  | Category                     | string | -                     |
| 7  | Category_type                | string | -                     |
| 8  | BMS Specific Medication flag | string | -                     |
| 9  | # patients                   | string | -                     |
| 10 | Total patients               | string | -                     |


7. RELATIONSHIPS MAPPING
7.1 Power BI Defined Relationships
| Relationship ID             | From Table                                  | From Column             | To Table                         | To Column | Type         | Cross-Filter | Notes                              |
|-----------------------------|---------------------------------------------|-------------------------|----------------------------------|-----------|--------------|--------------|------------------------------------|
| 7aa34c2c                    | Dashboard Refresh Date                      | ParameterTableKeyColumn | LocalDateTable_0feaa2be          | Date      | Many-to-One  | Single       | joinOnDateBehavior: datePartOnly    |
| AutoDetected_eb61f1d6       | Screen Config_Sheet1                        | Dataset Vendor          | Cost Analysis_Sheet1             | Dataset Vendor | One-to-Many | Both    | Auto-detected; bidirectional       |
| AutoDetected_44249bd9       | Use Case Analysis - Examples_Sheet1         | Attribute Name          | Table                            | Attribute Name | Many-to-One | Single  | Sort/lookup relationship           |
| fbd230bd                    | Outlier Patient Summary_5 Point Summary     | Category_type           | Table (2)                        | Category Type | Many-to-One | Single   | Sort order for Category and Units  |

7.2 Tableau vs. Power BI Relationship Approach
| Aspect                        | Tableau                                  | Power BI                                                       |
|-------------------------------|------------------------------------------|----------------------------------------------------------------|
| Cross-datasource relationships| Data blending                            | Defined model relationships                                    |
| Screen/filter cross-table     | Implicit via shared dimensions           | Explicit bidirectional relationship (Screen Config to Cost Analysis) |
| Sort order                    | Computed inline                          | Dedicated lookup Table (2) with sort key s                     |


8. SHOW/HIDE LOGIC PATTERN
8.1 Pattern Definition
-- Tableau
-- Step 1: Condition Calculated Field (integer)
if [Main Category]='<Category Name>' THEN 1 else 0 end

-- Step 2: Show/Hide Calculated Field (string measure)
if SUM([Condition <Category Name>])>=1
  THEN ""
  else "The data in the view is not available for the selected indication"
END

-- Power BI DAX
-- Step 1: Condition Calculated Column (int64)
'TableName'[Condition <Category Name>] =
    IF('TableName'[Main Category] = "<Category Name>", 1, 0)

-- Step 2: Show/Hide Measure (string)
[Show/Hide <Category Name>] =
    IF(
        SUM('TableName'[Condition <Category Name>]) >= 1,
        "",
        "The data in the view is not available for the selected indication"
    )

8.2 Patient Overall Show/Hide Measures
| Measure Name                  | Condition Column               | Main Category Value       |
|-------------------------------|--------------------------------|---------------------------|
| Show/Hide Top 10 Metric 1     | Condition Top 10 Metric 1      | "Top 10 Metric 1"         |
| Show/Hide Top 10 Metric 2     | Condition Top 10 Metric 2      | "Top 10 Metric 2"         |
| Show/Hide Top 10 Metric 3     | Condition Top 10 Metric 3      | "Top 10 Metric 3"         |
| Show/Hide Cohort Creation     | Condition Cohort Creation      | "Cohort Creation"         |
| Show/Hide Yearly Incidence    | Condition Yearly Incidence     | "Yearly Incidence"        |
| Show/Hide Coverage Metric 1   | Condition Coverage Metric 1    | "Age at first diagnosis"  |

8.3 Patient Cohort Show/Hide Measures
| Measure Name                              | Condition Column                          | Main Category Value              |
|-------------------------------------------|-------------------------------------------|----------------------------------|
| Show/Hide State                           | Condition States                          | "States"                         |
| Show/Hide Insurance Plan                  | Condition Insurance Plan                  | "Insurance Plan"                 |
| Show/Hide Gender                          | Condition Gender                          | "Gender"                         |
| Show/Hide Race                            | Condition Race                            | "Race"                           |
| Show/Hide Ethnicity                       | Condition Ethnicity                       | "Ethnicity"                      |
| Show/Hide Vitals                          | Condition Vitals                          | "Vitals"                         |
| Show/Hide Continuous Activity             | Condition Continuous Activity             | "Continuous Activity"            |
| Show/Hide Division                        | Condition Division                        | "Division"                       |
| Show/Hide Sdoh 1                          | Condition Sdoh 1                          | "SDOH Variable 1"                |
| Show/Hide Sdoh 2                          | Condition Sdoh 2                          | "SDOH Variable 2"                |
| Show/Hide Sdoh 3                          | Condition Sdoh 3                          | "SDOH Variable 3"                |
| Show/Hide Age at first diagnosis          | Condition Age at first diagnosis          | "Age at first diagnosis"         |
| Show/Hide Indication Specific Attribute 1 | Condition Indication Specific Attribute 1 | "Indication Specific Attribute 1"|
| Show/Hide Indication Specific Attribute 2 | Condition Indication Specific Attribute 2 | "Indication Specific Attribute 2"|
| Show/Hide Indication Specific Attribute 3 | Condition Indication Specific Attribute 3 | "Indication Specific Attribute 3"|
| Show/Hide Indication Specific Attribute 4 | Condition Indication Specific Attribute 4 | "Indication Specific Attribute 4"|
| Show/Hide Indication Specific Attribute 5 | Condition Indication Specific Attribute 5 | "Indication Specific Attribute 5"|
| Show/Hide Indication Specific Attribute 6 | Condition Indication Specific Attribute 6 | "Indication Specific Attribute 6"|
| Show/Hide Indication Specific Attribute 7 | Condition Indication Specific Attribute 7 | "Indication Specific Attribute 7"|
| Show/Hide Indication Specific Attribute 8 | Condition Indication Specific Attribute 8 | "Indication Specific Attribute 8"|
| Show/Hide Indication Specific Attribute 9 | Condition Indication Specific Attribute 9 | "Indication Specific Attribute 9"|

8.4 Cost Analysis Show/Hide
| Measure Name    | Condition Column | Main Category Value |
|-----------------|------------------|---------------------|
| Show/Hide Cost  | Condition Cost   | "Cost"              |


9. CONDITIONAL FLAG COLUMNS PATTERN
9.1 Full Condition Column Inventory
| Table                                       | PBI Condition Column                      | Trigger Value                    | lineageTag Reference                                        |
|---------------------------------------------|-------------------------------------------|----------------------------------|-------------------------------------------------------------|
| Patient Overall_Sheet1                      | Condition Top 10 Metric 1                 | "Top 10 Metric 1"                | Calculation_1470425322937380876                             |
| Patient Overall_Sheet1                      | Condition Top 10 Metric 2                 | "Top 10 Metric 2"                | Condition Top 10 Metric 1 (copy)_1470425322937765901        |
| Patient Overall_Sheet1                      | Condition Top 10 Metric 3                 | "Top 10 Metric 3"                | Condition Top 10 Metric 1 (copy) (copy)_1470425322937778190 |
| Patient Overall_Sheet1                      | Condition Cohort Creation                 | "Cohort Creation"                | Condition Payer Type (copy)_1107322603431878656             |
| Patient Overall_Sheet1                      | Condition Yearly Incidence                | "Yearly Incidence"               | Condition Top 10 Metric 1 (copy)_1470425322973397012        |
| Patient Overall_Sheet1                      | Condition Coverage Metric 1               | "Age at first diagnosis"         | Condition Top 10 Metric 1 (copy) (copy)_1470425322973409301 |
| patient_cohort_Sheet1                       | Condition States                          | "States"                         | Calculation_1800876918696902656                             |
| patient_cohort_Sheet1                       | Condition Insurance Plan                  | "Insurance Plan"                 | Condition States (copy)_1470425322979430429                 |
| patient_cohort_Sheet1                       | Condition Gender                          | "Gender"                         | Condition States (copy)_1470425322979946526                 |
| patient_cohort_Sheet1                       | Condition Race                            | "Race"                           | Condition States (copy) (copy)_1470425322979967007          |
| patient_cohort_Sheet1                       | Condition Ethnicity                       | "Ethnicity"                      | Condition States (copy)(copy)(copy)(copy)_...979995681      |
| patient_cohort_Sheet1                       | Condition Vitals                          | "Vitals"                         | Condition States (copy)(copy)(copy)_...979975200            |
| patient_cohort_Sheet1                       | Condition Division                        | "Division"                       | Condition States (copy)_1470425322968420370                 |
| patient_cohort_Sheet1                       | Condition Sdoh 1                          | "SDOH Variable 1"                | Condition States (copy)_1800876918704824322                 |
| patient_cohort_Sheet1                       | Condition Sdoh 2                          | "SDOH Variable 2"                | Condition Sdoh 1 (copy)_1800876918705508356                 |
| patient_cohort_Sheet1                       | Condition Sdoh 3                          | "SDOH Variable 3"                | Condition Sdoh 2 (copy)_1800876918705840134                 |
| patient_cohort_Sheet1                       | Condition Continuous Activity             | "Continuous Activity"            | Condition Sdoh 1 (copy)_1470425322977542169                 |
| patient_cohort_Sheet1                       | Condition Age at first diagnosis          | "Age at first diagnosis"         | Condition Division (copy)_1470425322978766874               |
| patient_cohort_Sheet1                       | Condition Indication Specific Attribute 1 | "Indication Specific Attribute 1"| ...923302917                                                |
| patient_cohort_Sheet1                       | Condition Indication Specific Attribute 2 | "Indication Specific Attribute 2"| ...923290628                                                |
| patient_cohort_Sheet1                       | Condition Indication Specific Attribute 3 | "Indication Specific Attribute 3"| ...923278339                                                |
| patient_cohort_Sheet1                       | Condition Indication Specific Attribute 4 | "Indication Specific Attribute 4"| ...923139074                                                |
| patient_cohort_Sheet1                       | Condition Indication Specific Attribute 5 | "Indication Specific Attribute 5"| ...923110401                                                |
| patient_cohort_Sheet1                       | Condition Indication Specific Attribute 6 | "Indication Specific Attribute 6"| ...923044864                                                |
| patient_cohort_Sheet1                       | Condition Indication Specific Attribute 7 | "Indication Specific Attribute 7"| Condition Sdoh 3 (copy)_1800876918706225160                 |
| patient_cohort_Sheet1                       | Condition Indication Specific Attribute 8 | "Indication Specific Attribute 8"| Condition Sdoh 3 (copy)(copy)_1800876918706253833           |
| patient_cohort_Sheet1                       | Condition Indication Specific Attribute 9 | "Indication Specific Attribute 9"| Condition Sdoh 3 (copy)(copy)(copy)_1800876918706266122     |
| Cost Analysis_Sheet1                        | Condition Cost                            | "Cost"                           | Calculation_1470425322993979433                             |


10. AUXILIARY AND SUPPORT TABLES
10.1 Table - Attribute Name Sort
| Column         | Type   | Purpose                                              |
|----------------|--------|------------------------------------------------------|
| Attribute Name | string | Links to Use Case Analysis - Examples_Sheet1         |
| (blank)        | int64  | Sort order key                                       |

Relationship: Use Case Analysis - Examples_Sheet1.[Attribute Name] → Table.[Attribute Name]
Source: Inline M query - compressed/base64 encoded static data
10.2 Table (2) - Category Type Sort
| Column        | Type   | Purpose                                                             |
|---------------|--------|---------------------------------------------------------------------|
| Category Type | string | Links to Outlier Patient Summary_5 Point Summary.[Category_type]    |
| s             | int64  | Sort order integer key                                              |

Relationship: Outlier Patient Summary_5 Point Summary.[Category_type] → Table (2).[Category Type]
Used by: Outlier Patient Summary_5 Point Summary.[sort] = RELATED('Table (2)'[s])
Source: Inline M query - compressed/base64 encoded static data
10.3 Date Tables (Auto-Generated)
| Table                                                   | Purpose                           | Linked To                                        |
|---------------------------------------------------------|-----------------------------------|--------------------------------------------------|
| DateTableTemplate_7c709509-6efb-4953-97e5-f58aa4764c5e  | Auto date table template          | Internal PBI engine                              |
| LocalDateTable_0feaa2be-9894-495b-a3f6-0dbc3437f1b2     | Local date hierarchy for Param 3  | Dashboard Refresh Date.ParameterTableKeyColumn   |


11. POWER BI MODEL METADATA
11.1 Model Settings
| Attribute                          | Value                                               |
|------------------------------------|-----------------------------------------------------|
| Model Name                         | 24363_DRWDS_Oncology_Capability Assessment- model   |
| Culture                            | en-US                                               |
| Source Query Culture               | en-US                                               |
| Default PBI Data Source Version    | powerBI_V3                                          |
| Legacy Redirects                   | Enabled                                             |
| Return Error Values as Null        | Enabled                                             |
| Time Intelligence Enabled          | Yes (__PBI_TimeIntelligenceEnabled = 1)             |
| Dev Mode                           | Yes (PBI_ProTooling = ["DevMode"])                  |

11.2 Table Load Order
| Order | Table Name                                  |
|-------|---------------------------------------------|
| 1     | Measures Table                              |
| 2     | Yearly Monthly                              |
| 3     | start date end date                         |
| 4     | Dashboard Refresh Date                      |
| 5     | Outlier Patient Summary_5 Point Summary     |
| 6     | Header Summary_Sheet1                       |
| 7     | patient_cohort_Sheet1                       |
| 8     | Patient Overall_Sheet1                      |
| 9     | Executive Summary_Sheet1                    |
| 10    | Use Case Analysis Updated_Sheet1            |
| 11    | Attribute Summary Updated_Sheet1            |
| 12    | Use Case Analysis - Examples_Sheet1         |
| 13    | Cost Analysis_Sheet1                        |
| 14    | Data Quality_Sheet1                         |
| 15    | Screen Config_Sheet1                        |
| 16    | DateTableTemplate (GUID)                    |
| 17    | LocalDateTable (GUID)                       |
| 18    | Table                                       |
| 19    | Table (2)                                   |


12. DATA SOURCE URLS
12.1 Per-Table Source URL Reference
| PBI Table                                   | SharePoint Excel File  |
|---------------------------------------------|------------------------|
| Patient Overall_Sheet1                      | Data Model 1.xlsx      |
| patient_cohort_Sheet1                       | Data Model 2.xlsx      |
| Attribute Summary Updated_Sheet1            | Data Model 3.xlsx      |
| Executive Summary_Sheet1                    | Data Model 4.xlsx      |
| Data Quality_Sheet1                         | Data Model 5.xlsx      |
| Use Case Analysis Updated_Sheet1            | Data Model 7.xlsx      |
| Outlier Patient Summary_5 Point Summary     | Data Model 8.xlsx      |
| Cost Analysis_Sheet1                        | Data Model 10.xlsx     |
| Screen Config_Sheet1                        | Data Model 10.xlsx     |
| Header Summary_Sheet1                       | Data Model 14.xlsx     |

Base Path:
https://sites.bms.com/sites/RWDSProjects-BMSFolder/Shared Documents/BMS Team Folder/Hyderabad - Internal BMS/02. Analytic Insights Projects/2026/Tableau Migration Excels/DRWDS_Oncology_Capability Assessment/


13. MIGRATION NOTES AND KNOWN ISSUES
13.1 Critical Migration Issues
| #  | Issue                               | Tableau Behavior                                                                   | Power BI Status                                              | Resolution                                                                      |
|----|-------------------------------------|------------------------------------------------------------------------------------|--------------------------------------------------------------|---------------------------------------------------------------------------------|
| 1  | Numeric columns stored as strings   | Total Patients, Number of Patients, # of Patient Captured typed as integer/real    | All stored as string in PBI                                  | Use IFERROR(VALUE(...),0) pattern in all aggregation measures                   |
| 2  | FIXED LOD to DAX RANKX              | RANK(SUM({FIXED [Category Type],[BMS Specific Medication flag]:...}),'desc')        | Requires RANKX + REMOVEFILTERS + KEEPFILTERS pattern         | Implemented in Rank measure                                                     |
| 3  | Tableau Table Calculations to DAX   | PctTotal table calculation                                                         | Requires explicit DIVIDE with correct filter context         | Placeholder measure created                                                     |
| 4  | Category Type Aliases               | "0.50"→"50%", "0.55"→"55%", etc.                                                  | Aliases not natively supported in PBI                        | Handle via Power Query replace or calculated column                             |
| 5  | BMS Medication Flag Aliases         | 0→"Other Brand Medication", 1→"BMS Brand Medication"                               | Not aliased in PBI                                           | Handle via calculated column or visual legend formatting                        |
| 6  | Non-Outlier Range (Cost)            | Referenced Min/Max Non Outlier Range columns                                       | PLACEHOLDER - columns not present in Data Model 10           | Fix when Cost Analysis source is confirmed                                      |
| 7  | Condition Division typo             | [Main Category]='Division'                                                         | PBI has "DIVIDEision" - copy-paste error                     | FIX REQUIRED: Change to "Division"                                              |
| 8  | Date parse format lost              | Refresh Date in patient_cohort has date-parse-format='yyyy/MM'                     | PBI stores as plain string                                   | Add explicit parsing step in M query if date operations are needed              |
| 9  | Tableau shape/mark encoding         | Funnel images, custom shape marks for cohort funnel                                | Not directly mirrored in PBI                                 | Implement as image URLs or conditional formatting icons                         |
| 10 | Color encoding                      | Per-bucket color maps for Category Type in patient_cohort                          | Not in current PBI tables                                    | Implement via DAX SWITCH color measures per visual                              |

13.2 Architecture Differences
| Aspect                | Tableau                                          | Power BI                                                                         |
|-----------------------|--------------------------------------------------|----------------------------------------------------------------------------------|
| Data refresh          | Hyper extract refresh via Tableau Server         | SharePoint Excel via Power BI Service scheduled refresh                          |
| Parameters            | Inline parameter control in workbook             | Slicer-bound parameter tables                                                    |
| Navigation            | Dashboard actions, button worksheets             | Next>/Prev navigation measures + bookmark actions                                |
| Show/Hide             | Floating container show/hide                     | DAX measure returns empty string; visual conditional visibility                  |
| Map visualizations    | Tableau built-in map with semantic role [State]  | PBI Map visual using explicit State Lat/State Lng columns (bypasses geocoding)   |

13.3 Successfully Migrated Patterns
| Pattern                                      | Status   |
|----------------------------------------------|----------|
| All if/THEN/else/end → IF()                  | Complete |
| String concatenation + → &                  | Complete |
| SUM()/MAX() aggregation pattern              | Complete |
| FIXED LOD RANK → RANKX                       | Complete |
| All Show/Hide condition pairs (22 pairs)     | Complete |
| Parameter tables (3 parameters)              | Complete |
| Category and Units concatenation             | Complete |
| Sort lookup tables                           | Complete |
| Geo columns for US state map                 | Complete |


14. COMPLETE FORMULA REFERENCE
-- % of Patients (patient_cohort and Patient Overall)
DIVIDE(
    SUMX('Table', IFERROR(VALUE('Table'[Number of Patients]), 0)),
    MAXX('Table', IFERROR(VALUE('Table'[Total Patients]), 0))
)

-- % of Patient Outlier
DIVIDE(
    CALCULATE(
        SUMX(
            'Outlier Patient Summary_5 Point Summary',
            IFERROR(VALUE('Outlier Patient Summary_5 Point Summary'[# of Outlier Patients]), 0)
        ),
        FILTER(
            'Outlier Patient Summary_5 Point Summary',
            'Outlier Patient Summary_5 Point Summary'[Patient Type] = "Outlier"
        )
    ),
    MAXX(
        'Outlier Patient Summary_5 Point Summary',
        IFERROR(VALUE('Outlier Patient Summary_5 Point Summary'[# of Patient Captured]), 0)
    )
)

-- Rank (replaces Tableau FIXED LOD RANK)
RANKX(
    ALL(
        'Patient Overall_Sheet1'[Category Type],
        'Patient Overall_Sheet1'[BMS Specific Medication flag]
    ),
    CALCULATE(
        SUM('Patient Overall_Sheet1'[Number of Patients]),
        REMOVEFILTERS('Patient Overall_Sheet1'),
        KEEPFILTERS(VALUES('Patient Overall_Sheet1'[Category Type])),
        KEEPFILTERS(VALUES('Patient Overall_Sheet1'[BMS Specific Medication flag]))
    ),
    , DESC, DENSE
)

-- Top 10 MEds
IF(
    OR(
        [Rank] <= 10,
        SUM('Patient Overall_Sheet1'[BMS Specific Medication flag]) = 1
    ),
    TRUE(), FALSE()
)

-- # Patients - Numeric
VAR _TextValue = TRIM('Attribute Summary Updated_Sheet1'[# Patients] & "")
VAR _CleanValue = SUBSTITUTE(_TextValue, ",", "")
RETURN
    IF(
        _TextValue = "" || _TextValue = "-",
        0,
        IFERROR(VALUE(_CleanValue), 0)
    )

-- % Patient - Final
IFERROR(VALUE('Attribute Summary Updated_Sheet1'[% Patients]), 0)

-- Data Quality Issue
'Data Quality_Sheet1'[DQ Issue Name] & " :" & 'Data Quality_Sheet1'[Details]

-- Category and Units
'Outlier Patient Summary_5 Point Summary'[Category_type]
    & " (in "
    & 'Outlier Patient Summary_5 Point Summary'[Units]
    & ")"

-- % Patients Bar Color
VAR _RawValue = [% of Patients (PO)]
VAR _Value = IF(_RawValue > 1, _RawValue / 100, _RawValue)
RETURN
    SWITCH(
        TRUE(),
        ISBLANK(_Value), "#EDEDED",
        _Value < 0.30,   "#E6E6E6",
        _Value < 0.40,   "#D8D2DD",
        _Value < 0.50,   "#C4B2CF",
        _Value < 0.60,   "#9F7EAD",
        _Value < 0.75,   "#7E5590",
        "#5B2D75"
    )

-- Fulfillment Color
SWITCH(
    TRUE(),
    SELECTEDVALUE('Use Case Analysis Updated_Sheet1'[Fulfillement Index])
        = "Can be fulfilled",     "#1a9850",
    SELECTEDVALUE('Use Case Analysis Updated_Sheet1'[Fulfillement Index])
        = "Can not be fulfilled", "#d73027",
    "#e67e22"
)

-- Use Case Desc Concat
CONCATENATEX(
    'Use Case Analysis Updated_Sheet1',
    "▸ " & 'Use Case Analysis Updated_Sheet1'[Use Case Description],
    UNICHAR(10),
    'Use Case Analysis Updated_Sheet1'[Use Case Description],
    ASC
)

-- Sum of Patients (PO) display
VAR _PatientCount =
    SUMX(
        'Patient Overall_Sheet1',
        IFERROR(VALUE('Patient Overall_Sheet1'[Number of Patients]), 0)
    )
RETURN "(N=" & FORMAT(_PatientCount, "#,##0") & ")"

-- INDEX() equivalent
RANKX(ALL('Measures Table'), 1)

-- Number of Patients Display (Data Quality)
VAR _Value = TRIM('Data Quality_Sheet1'[Number of Patients] & "")
VAR _CleanValue = SUBSTITUTE(_Value, ",", "")
RETURN
    IF(
        _Value = "" || UPPER(_Value) = "NA" || UPPER(_Value) = "N/A",
        "Null",
        FORMAT(IFERROR(VALUE(_CleanValue), 0), "#,##0")
    )

-- Show/Hide generic pattern
IF(
    SUM('TableName'[Condition <Name>]) >= 1,
    "",
    "The data in the view is not available for the selected indication"
)

-- States (patient_cohort)
IF(
    'patient_cohort_Sheet1'[Main Category] = "States",
    'patient_cohort_Sheet1'[Category Type],
    BLANK()
)

-- Category Type Sort (patient_cohort)
IFERROR(VALUE('patient_cohort_Sheet1'[Category Type]), 9999)

-- State Geo (patient_cohort)
SWITCH(
    'patient_cohort_Sheet1'[Category Type],
    "PR", "Puerto Rico, United States",
    "DC", "District of Columbia, United States",
    'patient_cohort_Sheet1'[Category Type] & ", United States"
)


KNOWLEDGE BASE SUMMARY
| Category                               | Count                                          |
|----------------------------------------|------------------------------------------------|
| Data Tables Migrated                   | 10                                             |
| Parameters Migrated                    | 3                                              |
| Measures (Total across all tables)     | 45+                                            |
| Calculated Columns (Total)             | 35+                                            |
| Show/Hide Measure Pairs                | 22                                             |
| Condition Flag Columns                 | 27                                             |
| Model Relationships                    | 4                                              |
| Auxiliary Lookup Tables                | 2                                              |
| Date Tables (auto-generated)           | 2                                              |
| Known Issues / Fixes Required          | 10                                             |
| Critical Fix (typo)                    | 1 ("DIVIDEision" must be corrected to "Division") |


Knowledge Base ID: K14
Created: 2026-06-12
Owner: BMS RWDS Oncology - Tableau Migration Team
Scope: Internal BMS Use Only
Source: 24363_drwds_oncology_capability_assessment.txt (Tableau .twb) + 25 Power BI PBIP output files