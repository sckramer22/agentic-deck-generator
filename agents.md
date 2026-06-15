# AGENTS.md

## Project

This project builds PowerPoint KPI reports from Worldpanel Insights Excel exports.

The V1 product is a structured internal web app that allows users to upload a Worldpanel Insights XLS/XLSX export and generate a PowerPoint report using provided Numerator/Worldpanel templates.

This is not a generic AI PowerPoint generator. It is a deterministic KPI report generator with a limited LLM narrative layer.

## V1 Goal

Generate a PowerPoint report that diagnoses sales growth or decline for a selected primary entity, such as a manufacturer, retailer, brand, or category.

The current sample use case is:

* Primary level: Manufacturer
* Primary entity: Coca-Cola
* Comparison level: Brand
* Secondary level: Category
* Input source: Worldpanel Insights Excel export
* Output: PowerPoint KPI report based on the provided sample slide template

## Core Principle

The system should be deterministic wherever possible.

The LLM must not decide:

* which calculations to perform
* which slides to create
* which chart types to use
* which rows or columns are correct
* which brands/categories rank highest
* where content goes in the PowerPoint

The LLM may only generate:

* slide titles framed as trends, conclusions, or key takeaways
* subtitles/supporting narrative explaining the data in plain English
* optional executive summary text in a later version

## Architecture

Use this flow:

```text
Worldpanel XLS/XLSX Export
    ↓
Config-driven Excel parser
    ↓
Canonical report JSON model
    ↓
Validation
    ↓
Deterministic slide plan
    ↓
LLM narrative generation
    ↓
PowerPoint renderer
```

Do not build direct Excel-to-PowerPoint spaghetti. The canonical JSON model is the contract between parsing, analytics, narrative generation, and slide creation.

## V1 Inputs

The app should support:

* a Worldpanel Insights Excel export
* a report configuration file
* a PowerPoint template file

The configuration should define:

* primary level name
* primary entity name
* comparison level name
* secondary level name
* sheet names
* metric mappings
* time period fields

Example:

```yaml
report:
  primary_level: Manufacturer
  primary_entity: Coca-Cola
  comparison_level: Brand
  secondary_level: Category

tabs:
  shopper_metrics: Shopper Metrics
  primary_by_month: Manufacturer by Month
  comparison_by_month: Brand by Month
  secondary_by_month: Category by Month
```

## V1 Excel Assumptions

The export is expected to contain:

1. A Shopper Metrics tab

   * used for KPI tree values
   * used for current/prior/change metrics
   * used for Level 1 and Level 2 summary tables

2. By-month tabs

   * used for line charts
   * may include primary entity trends
   * may include comparison-level trends
   * may include secondary-level trends

The parser should support different entity labels, such as Manufacturer, Retailer, Brand, Category, Banner, Product, or Store Format, but it should not try to interpret arbitrary Excel files.

This should be a config-driven parser, not a fully AI-inferred parser.

## Core Metrics

The report should support these KPI metrics:

* total sales value
* total sales volume
* total households
* household penetration
* repeat rate
* purchase frequency
* buy rate / spend per household
* volume per household
* spend per trip
* volume per trip
* units per trip
* spend per unit
* volume per unit

Not every report needs every metric. Missing metrics should be handled gracefully and flagged during validation.

## Slide Inventory

The provided PowerPoint template should be treated as the V1 slide inventory reference.

Expected slide types include:

1. Cover slide
2. KPI tree section divider
3. Level 1 value sales KPI tree
4. Level 1 volume sales KPI tree
5. KPI trend section divider
6. Metric-over-time slide for the primary entity
7. Level 2 KPI summary section divider
8. Value share comparison slide
9. Volume share comparison slide
10. Top 10 Level 2 KPI table
11. Level 2 metric-over-time slide for top entities

Use the template layouts and placeholders where possible. Do not invent new layouts unless explicitly needed.

## Narrative Generation

The LLM receives structured JSON only. It should not receive raw Excel tables unless absolutely necessary.

For each slide requiring narrative, generate:

```json
{
  "title": "Main takeaway written as a conclusion",
  "subtitle": "Short supporting explanation using specific metric evidence"
}
```

Good title example:

```text
Coca-Cola Growth Was Driven Primarily by Expanded Household Reach
```

Good subtitle example:

```text
Household penetration increased while purchase frequency remained relatively flat, indicating growth came more from reaching new buyers than from higher engagement among existing buyers.
```

Bad title example:

```text
Household Penetration Over Time
```

Bad subtitle example:

```text
This slide shows household penetration over time.
```

## Validation Philosophy

Step 1 should validate the canonical JSON before any PowerPoint is generated.

Validation should confirm:

* required sheets are present
* required entity levels are found
* primary entity is found
* required metrics are found or clearly listed as missing
* monthly tabs contain parseable time periods
* current/prior/change fields are numeric where expected
* no duplicate entity/metric combinations exist unless expected
* output JSON conforms to the schema

If validation fails, return clear errors and warnings. Do not silently continue.

## First Milestone

Build only the Excel parser and canonical JSON writer.

Done means:

* the sample XLS/XLSX can be parsed
* a canonical JSON file is written
* validation passes
* tests prove key fields are correct
* no PowerPoint generation yet

## Suggested Repo Structure

```text
src/
  config/
    default_report_config.yaml

  parser/
    excel_parser.ts
    shopper_metrics_parser.ts
    monthly_parser.ts

  model/
    report_model.ts
    report_schema.json

  validation/
    validate_report.ts

  narrative/
    prompt_builder.ts

  ppt/
    slide_plan.ts
    renderer.ts

tests/
  fixtures/
    sample_export.xlsx
    expected_report_minimal.json

  parser.test.ts
  validation.test.ts
```

## Coding Rules

* Keep parsing separate from slide generation.
* Keep validation separate from parsing.
* Use typed models where possible.
* Avoid hardcoded references to Coca-Cola except in test fixtures.
* Avoid hardcoded references to Manufacturer except in config examples.
* Do not call the LLM during parser tests.
* Do not generate PowerPoint until the canonical JSON model is validated.

## Step 1 Done Criteria

Step 1 is complete when running the parser on the sample export produces a JSON file with:

* report metadata
* primary entity details
* KPI tree metrics
* comparison-level entities
* secondary-level entities if present
* monthly time series
* validation status
* warnings and errors

The output should be human-readable and stable enough to review in Git.
