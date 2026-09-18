# DISEASE_OUTBREAK_TRACKER
An educational, Python-based decision-support prototype that organizes simulated disease outbreak records, validates data quality, calculates key public-health metrics, classifies outbreaks by risk level, and supports search, monitoring, and summary reporting.

## Problem Statement

Public-health teams across Africa often work with outbreak data that is scattered, inconsistently recorded, and slow to review — a delay of even a few days between a case being reported and an analyst spotting a dangerous pattern can mean the difference between a contained cluster and a full-blown epidemic. Small teams and training programs rarely have easy access to a lightweight tool that can take raw case-count records, flag data-quality problems, and immediately surface which outbreaks need urgent attention. This project addresses that gap at a prototype scale: given a list of outbreak records, how can a program consistently and transparently tell an analyst which ones matter most right now?

## Project Objective

The objective of this project is to build a Python-based prototype that can **store, validate, analyse, search, and summarize** simulated disease outbreak records. The program classifies each valid record according to predefined, transparent risk rules (Low / Moderate / High / Critical) and generates automated alerts for records that need further review — giving a small team a repeatable, explainable first pass over outbreak data before human experts take over.

> **Note:** All data used in this project is **simulated** and created strictly for educational purposes. The figures should not be interpreted as real epidemiological statistics, and the risk/alert thresholds are classroom rules, not official WHO or Africa CDC classifications.

## Dataset

The dataset contains **23 simulated disease outbreak records** from different African countries and regions:
- **15** original records
- **3** intentionally invalid records (used to test the validation logic)
- **5** student-created records (one of which is also intentionally invalid)

All figures are fictional and were generated only to exercise the program's logic — they do not represent real outbreaks.

## Variables

| Variable | Description |
|---|---|
| `outbreak_id` | Unique identifier for the record (e.g. `OUT001`) |
| `country` | Country where the outbreak was reported |
| `state_region` | State or region within the country |
| `disease` | Disease associated with the outbreak (e.g. Cholera, Measles) |
| `suspected_cases` | Number of suspected cases |
| `confirmed_cases` | Number of confirmed cases |
| `deaths` | Number of deaths |
| `recovered` | Number of recovered cases |
| `active_cases` | Number of currently active cases |
| `report_date` | Date the record was reported (`YYYY-MM-DD`) |

**Full codebook (all 23 records):**

`OUT016`, `OUT017`, `OUT018`, and `OUT023` are intentionally invalid — they exist to exercise the validation logic (see Methodology below) and are excluded from metrics, risk classification, and summaries.

| outbreak_id | country | state_region | disease | suspected_cases | confirmed_cases | deaths | recovered | active_cases | report_date |
|---|---|---|---|---|---|---|---|---|---|
| OUT001 | Nigeria | Lagos | Cholera | 250 | 180 | 12 | 130 | 38 | 2026-08-01 |
| OUT002 | Ghana | Greater Accra | Measles | 150 | 100 | 4 | 75 | 21 | 2026-08-02 |
| OUT003 | Kenya | Nairobi | Cholera | 320 | 240 | 14 | 170 | 56 | 2026-08-03 |
| OUT004 | Uganda | Kampala | Yellow Fever | 90 | 60 | 3 | 45 | 12 | 2026-08-04 |
| OUT005 | Nigeria | Kano | Measles | 600 | 520 | 24 | 410 | 86 | 2026-08-05 |
| OUT006 | Sierra Leone | Western Area | Cholera | 420 | 310 | 16 | 220 | 74 | 2026-08-06 |
| OUT007 | South Africa | Gauteng | Influenza | 300 | 180 | 5 | 150 | 25 | 2026-08-07 |
| OUT008 | Liberia | Montserrado | Lassa Fever | 130 | 105 | 13 | 70 | 22 | 2026-08-08 |
| OUT009 | Senegal | Dakar | Measles | 75 | 50 | 2 | 38 | 10 | 2026-08-09 |
| OUT010 | Nigeria | Rivers | Yellow Fever | 210 | 155 | 9 | 110 | 36 | 2026-08-10 |
| OUT011 | Cameroon | Centre | Cholera | 700 | 580 | 27 | 420 | 133 | 2026-08-11 |
| OUT012 | Rwanda | Kigali | Measles | 40 | 25 | 1 | 20 | 4 | 2026-08-12 |
| OUT013 | Ghana | Ashanti | Cholera | 280 | 210 | 11 | 145 | 54 | 2026-08-13 |
| OUT014 | Zambia | Lusaka | Cholera | 180 | 120 | 6 | 90 | 24 | 2026-08-14 |
| OUT015 | Ethiopia | Addis Ababa | Measles | 500 | 350 | 18 | 260 | 72 | 2026-08-15 |
| OUT016 *(invalid)* | Nigeria | Kaduna | Cholera | 100 | 150 | 5 | 80 | 65 | 2026-08-16 |
| OUT017 *(invalid)* | Ghana | Volta | Measles | 100 | 70 | 80 | 40 | 10 | 2026-08-17 |
| OUT018 *(invalid)* | Kenya | Mombasa | Cholera | 200 | 150 | 5 | 100 | 200 | 2026-08-18 |
| OUT019 | Botswana | Gaborone | Influenza | 35 | 18 | 0 | 14 | 4 | 2026-08-19 |
| OUT020 | Tanzania | Dar es Salaam | Cholera | 360 | 260 | 9 | 190 | 61 | 2026-08-20 |
| OUT021 | Nigeria | Oyo | Measles | 230 | 170 | 7 | 120 | 43 | 2026-08-21 |
| OUT022 | Namibia | Khomas | Yellow Fever | 140 | 95 | 3 | 70 | 22 | 2026-08-22 |
| OUT023 *(invalid)* | Uganda | Gulu | Lassa Fever | 80 | 110 | 6 | 50 | 54 | 2026-08-23 |

## Methodology

The program processes data through a single, repeatable pipeline:

```
Data Input
↓
Validation
↓
Metric Calculation
↓
Risk Classification
↓
Search & Monitoring
↓
Summary
↓
Insights
```

### 1. Data Input

Every outbreak record is stored as a dictionary inside a single list, so the whole dataset can be passed around and updated as one object.

```python
outbreaks = [
    {"outbreak_id": "OUT001", "country": "Nigeria", "state_region": "Lagos",
     "disease": "Cholera", "suspected_cases": 250, "confirmed_cases": 180,
     "deaths": 12, "recovered": 130, "active_cases": 38, "report_date": "2026-08-01"},
    # ... 22 more records
]
```

### 2. Validation

Every record is checked against the project's data-quality rules before it can be used anywhere else in the pipeline. A record is only `is_valid()` once its error list comes back empty.

```python
def validation_errors(record):
    """Return a list of every project-rule violation in one record."""
    errors = []
    numeric_fields = ["suspected_cases", "confirmed_cases", "deaths", "recovered", "active_cases"]
    index = 0
    while index < len(numeric_fields):
        field = numeric_fields[index]
        if record[field] < 0:
            errors.append(field.replace("_", " ").title() + " cannot be negative.")
        index += 1
    if record["confirmed_cases"] > record["suspected_cases"]:
        errors.append("Confirmed cases exceed suspected cases.")
    if record["deaths"] > record["confirmed_cases"]:
        errors.append("Deaths exceed confirmed cases.")
    if record["recovered"] > record["confirmed_cases"]:
        errors.append("Recovered cases exceed confirmed cases.")
    if record["active_cases"] > record["confirmed_cases"]:
        errors.append("Active cases exceed confirmed cases.")
    return errors


def is_valid(record):
    return len(validation_errors(record)) == 0
```

### 3. Metric Calculation

For every valid record, four rates are calculated safely — dividing by zero simply returns `None` instead of crashing the program.

```python
def calculate_metrics(record):
    """Calculate required rates safely; return None for unavailable denominators."""
    suspected = record["suspected_cases"]
    confirmed = record["confirmed_cases"]
    confirmation_rate = None
    fatality_rate = None
    recovery_rate = None
    active_rate = None
    if suspected > 0:
        confirmation_rate = confirmed / suspected * 100
    if confirmed > 0:
        fatality_rate = record["deaths"] / confirmed * 100
        recovery_rate = record["recovered"] / confirmed * 100
        active_rate = record["active_cases"] / confirmed * 100
    return {
        "confirmation_rate": confirmation_rate,
        "case_fatality_rate": fatality_rate,
        "recovery_rate": recovery_rate,
        "active_case_rate": active_rate,
    }
```

### 4. Risk Classification

Each valid record is assigned a priority of **LOW**, **MODERATE**, **HIGH**, or **CRITICAL** based on confirmed cases, deaths, and case fatality rate — and the function returns *why* it reached that verdict, not just the verdict itself.

```python
def classify_risk(record):
    """Return the analytical priority and transparent reason(s)."""
    metrics = calculate_metrics(record)
    fatality_rate = metrics["case_fatality_rate"]
    reasons = []
    critical_volume = record["confirmed_cases"] >= CRITICAL_CONFIRMED_THRESHOLD and record["deaths"] >= CRITICAL_DEATH_THRESHOLD
    critical_fatality = fatality_rate is not None and fatality_rate >= CRITICAL_FATALITY_RATE and record["confirmed_cases"] >= CRITICAL_FATALITY_MIN_CASES
    if critical_volume or critical_fatality:
        if critical_volume:
            reasons.append("Confirmed cases and deaths reached the critical volume thresholds.")
        if critical_fatality:
            reasons.append("Case fatality rate and confirmed cases reached the critical severity thresholds.")
        return "CRITICAL", reasons
    if record["confirmed_cases"] >= HIGH_CONFIRMED_THRESHOLD or record["deaths"] >= HIGH_DEATH_THRESHOLD:
        if record["confirmed_cases"] >= HIGH_CONFIRMED_THRESHOLD:
            reasons.append("Confirmed cases reached the high-case threshold.")
        if record["deaths"] >= HIGH_DEATH_THRESHOLD:
            reasons.append("Reported deaths reached the high-death threshold.")
        return "HIGH", reasons
    if record["confirmed_cases"] >= MODERATE_CONFIRMED_THRESHOLD:
        reasons.append("Confirmed cases reached the moderate-case threshold.")
        return "MODERATE", reasons
    reasons.append("The record did not reach the moderate, high, or critical thresholds.")
    return "LOW", reasons
```

### 5. Search & Monitoring

Records can be searched by any field (country or disease), and every match is shown with its risk classification. High-priority records can also be filtered on demand, reusing the exact same `classify_risk()` logic.

```python
def search_records(records, field, label):
    """Search all records (valid and invalid) by an exact, case-insensitive
    match on the given field (e.g. field="country", label="Country")."""
    term = input(f"Enter {label.lower()} to search for: ").strip()
    if term == "":
        print(f"\n{label} cannot be blank.")
        return

    print(f"\n{label.upper()} SEARCH RESULTS: {term.title()}")
    print("-" * 62)
    count = 0
    index = 0
    while index < len(records):
        record = records[index]
        if record[field].lower() == term.lower():
            display_record(record, show_risk=True)
            count += 1
        index += 1

    print(f"\nTotal matching records: {count}")
    if count == 0:
        print(f"No records found for that {label.lower()}.")


def show_high_priority(records):
    """Show every valid record classified HIGH or CRITICAL, using the same
    classify_risk() rules as the rest of the program."""
    print("\n=== HIGH-PRIORITY OUTBREAKS (CLOSE MONITORING) ===")
    found = False
    index = 0
    while index < len(records):
        record = records[index]
        if is_valid(record):
            risk, reasons = classify_risk(record)
            if risk in ("HIGH", "CRITICAL"):
                display_record(record, show_risk=True)
                found = True
        index += 1
    if not found:
        print("No HIGH or CRITICAL records found.")
```

### 6. Summary

`build_summary()` aggregates every valid record into totals, disease/country breakdowns, and risk-level counts in a single pass.

```python
def build_summary(records):
    valid = valid_records(records)
    totals = {"suspected": 0, "confirmed": 0, "deaths": 0, "recovered": 0, "active": 0}
    disease_records = {}
    disease_confirmed = {}
    disease_deaths = {}
    country_records = {}
    country_confirmed = {}
    risk_counts = {"LOW": 0, "MODERATE": 0, "HIGH": 0, "CRITICAL": 0}
    highest_cfr_id = "N/A"
    highest_cfr = -1.0
    index = 0
    while index < len(valid):
        record = valid[index]
        totals["suspected"] += record["suspected_cases"]
        totals["confirmed"] += record["confirmed_cases"]
        totals["deaths"] += record["deaths"]
        totals["recovered"] += record["recovered"]
        totals["active"] += record["active_cases"]
        increment_count(disease_records, record["disease"])
        increment_count(disease_confirmed, record["disease"], record["confirmed_cases"])
        increment_count(disease_deaths, record["disease"], record["deaths"])
        increment_count(country_records, record["country"])
        increment_count(country_confirmed, record["country"], record["confirmed_cases"])
        risk, reasons = classify_risk(record)
        risk_counts[risk] += 1
        cfr = calculate_metrics(record)["case_fatality_rate"]
        if cfr is not None and cfr > highest_cfr:
            highest_cfr = cfr
            highest_cfr_id = record["outbreak_id"]
        index += 1
    most_disease, most_disease_count = largest_key(disease_records)
    top_disease_cases, top_disease_cases_value = largest_key(disease_confirmed)
    top_disease_deaths, top_disease_deaths_value = largest_key(disease_deaths)
    top_country_cases, top_country_cases_value = largest_key(country_confirmed)
    top_country_records, top_country_records_value = largest_key(country_records)
    return {
        "total_records": len(records), "valid_records": len(valid), "invalid_records": len(records) - len(valid),
        "totals": totals, "risk_counts": risk_counts,
        "most_disease": most_disease, "most_disease_count": most_disease_count,
        "top_disease_cases": top_disease_cases, "top_disease_cases_value": top_disease_cases_value,
        "top_disease_deaths": top_disease_deaths, "top_disease_deaths_value": top_disease_deaths_value,
        "top_country_cases": top_country_cases, "top_country_cases_value": top_country_cases_value,
        "top_country_records": top_country_records, "top_country_records_value": top_country_records_value,
        "highest_cfr_id": highest_cfr_id, "highest_cfr": highest_cfr,
        "disease_records": disease_records, "country_confirmed": country_confirmed,
    }
```

### 7. Insights

`generate_summary()` turns the aggregated dictionary above into a plain-language report — the point where numbers become an answer to "what actually happened in this data?"

```python
def generate_summary(records):
    summary = build_summary(records)
    totals = summary["totals"]
    risks = summary["risk_counts"]
    print("\n" + "=" * 62)
    print("DISEASE OUTBREAK SUMMARY - VALID RECORDS ONLY")
    print("=" * 62)
    print(f'Total Records: {summary["total_records"]}')
    print(f'Valid Records: {summary["valid_records"]}')
    print(f'Invalid Records Excluded from Totals: {summary["invalid_records"]}')
    print(f'Total Suspected Cases: {totals["suspected"]:,}')
    print(f'Total Confirmed Cases: {totals["confirmed"]:,}')
    print(f'Total Deaths: {totals["deaths"]:,}')
    print(f'Total Recovered: {totals["recovered"]:,}')
    print(f'Total Active Cases: {totals["active"]:,}')
    print(f'Low: {risks["LOW"]} | Moderate: {risks["MODERATE"]} | High: {risks["HIGH"]} | Critical: {risks["CRITICAL"]}')
    print(f'High/Critical Records: {risks["HIGH"] + risks["CRITICAL"]}')
    print(f'Most Reported Disease: {summary["most_disease"]} ({summary["most_disease_count"]} records)')
    print(f'Disease With Highest Confirmed Cases: {summary["top_disease_cases"]} ({summary["top_disease_cases_value"]:,})')
    print(f'Disease With Highest Deaths: {summary["top_disease_deaths"]} ({summary["top_disease_deaths_value"]:,})')
    print(f'Country With Highest Confirmed Cases: {summary["top_country_cases"]} ({summary["top_country_cases_value"]:,})')
    print(f'Country With Most Records: {summary["top_country_records"]} ({summary["top_country_records_value"]})')
    print(f'Highest Case Fatality Rate: {summary["highest_cfr_id"]} ({summary["highest_cfr"]:.2f}%)')
    print("\nResponsible interpretation: Results describe simulated training data only.")
```

## Key Findings

Based on the simulated 23-record dataset:

1. The dataset contains 23 records, of which 19 are valid and 4 are invalid.
2. The valid records contain 3,728 confirmed cases and 184 deaths.
3. Cholera is the most reported disease, appearing in 7 valid records.
4. Cholera also has the highest total number of confirmed cases.
5. Nigeria has the highest number of confirmed cases, with 1,025.
6. Nine valid records are classified as High or Critical.
7. Three valid records are classified as Critical.
8. Four records are excluded from analysis because they fail validation.

These findings describe only the simulated classroom dataset and should not be read as real-world epidemiological conclusions.

## Limitations

- The dataset is simulated and relatively small — it does not reflect the scale or noise of real surveillance data.
- The system does not connect to a live public-health database or reporting feed (e.g. IDSR/DHIS2).
- The risk and alert thresholds are educational rules, not official WHO or Africa CDC classifications.
- Population size and historical outbreak trends are not included, so results cannot be adjusted per-capita.
- The system does not forecast or predict future outbreaks — it only classifies current data.
- All alerts and classifications require interpretation and review by qualified public-health professionals before any action is taken.

## Future Improvements

- Connect the program to a real, regularly updated data source (e.g. an IDSR or DHIS2 feed).
- Add visual charts and a dashboard for trend monitoring over time.
- Include population data so rates can be compared fairly across regions.
- Add historical records to support trend and time-series analysis.
- Allow validated records to be saved permanently (e.g. to a database or file).
- Have public-health professionals formally review and calibrate the risk and alert thresholds.
-
