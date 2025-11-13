# AI Agent Instructions for PRODTECH_INVESTMENTS

## Project Overview
This is a data analysis workspace for ProdTech team demographics, performance tracking, and investment analysis at DigitalRoute. The primary workflow involves Jupyter notebook analysis of employee data with standardized reporting outputs.

## Data Architecture

### Core Data Sources
- **Employee Data**: `data/*-2025-11-03-PT-all-employees.xlsx` - Historical snapshots with dated filenames
- **Cost Centers**: `data/cost-centers.xlsx` - Maps cost center codes to teams/products (CC, CC_name, product)
- **Currency data**: `data/reference-forex-etc.xlsx` - Maps countries to currency codes and exchange rates

### Data Schema Patterns

#### Core Employee Data Fields
- **Personal Info**: `fname`, `lname`, `gender`, `dob` (date of birth), `age` (calculated)
- **Employment**: `start`, `end`, `title`, `function`, `team`, `dept`, `manager`, `contract`, `ismanager`
- **Location**: `country`, `city`
- **Financial**: `salary` (base), `TTC` (total target compensation), employee cost calculations
- **Administrative**: `CC` (cost center), `employee_id`

## Dataframe Architecture

### Primary Dataframes

#### `df` - Raw Employee Data
- **Source**: Excel file loaded from `data/*.xlsx` 
- **Content**: Complete employee records with original column names
- **Key Operations**: Initial data loading, column validation, data type conversions
- **Transforms**: Column renaming, date parsing, cost center string conversion

#### `df_ext` - Enhanced Employee Data  
- **Source**: Enriched version of `df`
- **Content**: Original data plus calculated fields (age, tenure, etc.)
- **Key Fields Added**: `age` (calculated from DOB), enhanced data types
- **Purpose**: Base dataframe for all downstream analysis

#### `df_active` - Current Employees
- **Source**: Filtered from `df_ext`
- **Filter Logic**: `(end.isnull()) | (end > today)` 
- **Content**: Only currently employed staff (active + future starts)
- **Usage**: Primary dataframe for demographic analysis, headcount reporting
- **Key Analytics**: Team composition, location distribution, function breakdown

#### `ref_cc` - Cost Center Reference
- **Source**: `data/cost-centers.xlsx`
- **Content**: Maps cost center codes to team/product names
- **Schema**: `CC` (cost center code), `CC_name` (team name), `product`
- **Join Pattern**: Used to enrich employee data with team/product information

#### `ref` - Currency Reference Data
- **Source**: `data/reference-forex-etc.xlsx` 
- **Content**: Country to currency mapping, exchange rates
- **Purpose**: Financial calculations, cost normalization across countries

### Analysis-Specific Dataframes

#### `df_joiners_leavers` - Yearly Movement Tracking
- **Structure**: Year as index, columns: `['joiners', 'leavers', 'increase', 'total_start', 'total_end']`
- **Purpose**: Annual headcount movement analysis (2020-2025)
- **Calculations**: Net changes, year-over-year growth trends

#### `df_joiners_leavers_monthly` - Monthly Movement Tracking  
- **Structure**: Month as index, columns: `['joiners', 'leavers', 'increase']`
- **Purpose**: Current year monthly hiring/attrition patterns
- **Scope**: Detailed monthly breakdown for current year

#### Financial Summary Dataframes
- **`df_by_function`**: Salary/cost aggregations by function (Engineering, PM, etc.)
- **`df_by_country`**: Geographic cost distribution and headcount
- **`df_cc_summary`**: Cost center rollups with reference data joins

#### Specialized Analysis Dataframes
- **Team-specific filters**: Engineering teams, R&D groups, management hierarchies
- **Milestone tracking**: Work anniversary analysis, tenure-based segmentation  
- **Outlier analysis**: Combined salary/tenure outliers using `pd.concat()`

### Common Data Patterns

#### Join Operations
```python
# Cost center enrichment
df_cc_summary.join(ref_cc.set_index('CC'), on='CC', how='left')

# Country-level aggregations  
df_by_country.join(employees_by_country)
```

#### Filtering Patterns
```python
# Active employees only
df_active = df_ext[(df_ext.end.isnull()) | (df_ext['end'] > today)]

# Year-based filtering for joiners/leavers
joiners = df[df.start.dt.year.eq(year)]
leavers = df[df.end.dt.year.eq(year)]
```

#### Aggregation Patterns
- Function-level: `groupby('function')` for demographic breakdowns
- Geographic: `groupby(['country', 'city'])` for location analysis  
- Temporal: Date-based grouping for trend analysis
- Cost Center: Financial rollups by team/product alignment

## File Naming Conventions

### Strict Naming Requirements
- **Data Files**: `YYYY-MM-DD-description.{csv,xlsx}` format
- **Report Outputs**: `YYYY-MM-DD-reportname.{html,pdf}` in `/reports/` directory
- **Multiple Versions**: Use suffixes like `-v2`, `-ALL`, `-FULL`, `-demed` for variants

### Report Categories


## Development Workflow

### Jupyter Notebook Patterns
- Export final reports as HTML/PDF to `/reports/`
- Use data from `/data/` directory with dated file selection
- HTML exports include embedded styling for standalone viewing

### Data Privacy & Security
- All data files are gitignored for privacy
- Employee data contains sensitive information (salaries, performance ratings)

## Key Analysis Patterns


## Common Tasks

### When Creating New Analysis
1. Select latest data file with `YYYY-MM-DD` prefix
2. Use descriptive report names following naming convention
3. Export to `/reports/` with current date prefix
4. Generate both HTML (for viewing) and PDF (for distribution) when needed

### When Working with Employee Data
- Join demographics with cost center data on cost center codes
- Handle missing/null performance data gracefully
- Respect data privacy in any outputs or exports

### Report Generation Best Practices
- Include date ranges in analysis descriptions
- Use consistent team/function categorization
- Generate audience-specific views (full vs. filtered)
- Maintain historical comparison capabilities