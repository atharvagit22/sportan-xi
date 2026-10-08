# End-to-End Cricket Data Analytics Project

## Table of Contents

1. [Overview](#overview)
2. [Business Context and Objectives](#business-context-and-objectives)
3. [Pipeline Architecture](#pipeline-architecture)
4. [Repository Structure](#repository-structure)
5. [Data Sources](#data-sources)
6. [Data Model](#data-model)
7. [Technology Stack](#technology-stack)
8. [Data Engineering Workflow](#data-engineering-workflow)
9. [Best 11 Selection Methodology](#best-11-selection-methodology)
10. [Visualization Layer](#visualization-layer)
11. [Assumptions and Limitations](#assumptions-and-limitations)
12. [Future Enhancements](#future-enhancements)
13. [Author](#author)

---

## Overview

This project implements an end-to-end data analytics pipeline for T20 World Cup cricket data. The pipeline covers data acquisition, cleaning, transformation, analysis, and rule-based team selection. Raw data is collected using Bright Data scrapers and ESPN Cricinfo scorecards, stored as JSON, normalized into a dimensional CSV model, and analyzed with Pandas inside a Jupyter Notebook.

The final output is an optimized "Best 11" team, selected programmatically through conditional, role-based logic applied to batting and bowling performance data. All analysis and visualization is performed in Python using `matplotlib`, `seaborn`, and `plotly`. No Power BI or other external business intelligence tooling is used.

## Business Context and Objectives

The project is framed around a scoping brief titled "Project Sportan". The brief states that the opposition's strengths and weaknesses are unknown, and requests the best possible 11 players, subject to two team-level performance targets:

1. The team should be able to score at least 180 runs on average.
2. The team should be able to defend 150 runs on average.

Objectives:

- Acquire structured T20 World Cup match, batting, bowling, and player data from public sources.
- Persist raw extractions as JSON to preserve source fidelity.
- Clean, validate, and transform raw data into analysis-ready dimension and fact tables.
- Compute player-level performance metrics for batting and bowling.
- Apply deterministic, role-based selection criteria to construct the Best 11 team.
- Present findings through reproducible, code-driven visualizations.

## Pipeline Architecture

```text
Data Acquisition        Raw Storage          Transformation          Analysis and Selection
----------------        -----------          --------------          ----------------------
Bright Data scrapers -> t20_json_files/  ->  Pandas cleaning     ->  Metric computation
ESPN Cricinfo           (raw JSON)           and normalization       Role-based filtering
scorecards                                   |                       Best 11 selection
                                             v                       |
                                        t20_csv_files/               v
                                        (dimension and fact      Visualization
                                         tables)                 (matplotlib, seaborn, plotly)
```

1. **Acquisition:** Bright Data scrapers and ESPN Cricinfo scorecards provide match results, batting summaries, bowling summaries, and player information.
2. **Raw storage:** Extracted payloads are stored unmodified as JSON files.
3. **Transformation:** JSON is loaded into Pandas, cleaned, typed, and exported as CSV tables organized as dimensions and facts.
4. **Analysis:** Aggregated performance metrics are computed per player and per role.
5. **Selection:** Conditional logic evaluates players against role-specific criteria to produce the Best 11.
6. **Visualization:** Results are rendered using Python plotting libraries.

## Repository Structure

```text
cricket analysis project/
│
├── t20_csv_files/
│   ├── dim_match_summary.csv
│   ├── dim_players.csv
│   ├── dim_players_no_images.csv
│   ├── fact_bating_summary.csv
│   └── fact_bowling_summary.csv
│
├── t20_json_files/
│   ├── t20_wc_batting_summary.json
│   ├── t20_wc_bowling_summary.json
│   ├── t20_wc_match_results.json
│   └── t20_wc_player_info.json
│
└── cricket_analysis.ipynb
```

### Component Descriptions

| Path | Description |
|------|-------------|
| `t20_json_files/` | Raw data extracted from the source systems, stored as JSON. |
| `t20_json_files/t20_wc_batting_summary.json` | Raw batting scorecard data. |
| `t20_json_files/t20_wc_bowling_summary.json` | Raw bowling scorecard data. |
| `t20_json_files/t20_wc_match_results.json` | Raw match-level results. |
| `t20_json_files/t20_wc_player_info.json` | Raw player profile information. |
| `t20_csv_files/` | Cleaned and transformed tables used for analysis. |
| `t20_csv_files/dim_match_summary.csv` | Match dimension table. |
| `t20_csv_files/dim_players.csv` | Player dimension table, including image references. |
| `t20_csv_files/dim_players_no_images.csv` | Player dimension table with image fields removed. |
| `t20_csv_files/fact_bating_summary.csv` | Batting fact table. The filename spelling is retained as it appears in the repository. |
| `t20_csv_files/fact_bowling_summary.csv` | Bowling fact table. |
| `cricket_analysis.ipynb` | Jupyter Notebook containing cleaning, analysis, selection logic, and visualizations. |

## Data Sources

| Source | Purpose |
|--------|---------|
| Bright Data scrapers | Automated extraction of T20 World Cup data at scale. |
| ESPN Cricinfo scorecards | Match scorecards providing batting, bowling, and result information. |

Users are responsible for complying with the terms of service of each data provider when collecting or redistributing data.

## Data Model

The CSV layer follows a dimensional modeling approach.

**Dimension tables**

- `dim_match_summary`: one record per match, describing match-level attributes and results.
- `dim_players`: one record per player, describing player attributes such as name, team, and bowling style.
- `dim_players_no_images`: a lightweight variant of `dim_players` without image fields, intended for faster loading and analysis.

**Fact tables**

- `fact_bating_summary`: batting performance per player per match.
- `fact_bowling_summary`: bowling performance per player per match.

Fact tables relate to dimension tables through match and player identifiers. Refer to the notebook for the exact column definitions and join keys used.

## Technology Stack

| Category | Tools |
|----------|-------|
| Language | Python 3 |
| Data manipulation | Pandas |
| Visualization | matplotlib, seaborn, plotly |
| Environment | Jupyter Notebook |
| Data acquisition | Bright Data scrapers, ESPN Cricinfo scorecards |
| Data formats | JSON (raw), CSV (processed) |

## Data Engineering Workflow

1. **Ingestion.** Load the four raw JSON files from `t20_json_files/` into Pandas DataFrames.
2. **Flattening.** Convert nested JSON structures into tabular form.
3. **Cleaning.**
   - Remove duplicate records.
   - Standardize player and team names.
   - Handle missing and malformed values.
   - Correct data types for numeric, categorical, and date fields.
4. **Modeling.** Separate attributes into dimension tables and measurable events into fact tables.
5. **Export.** Write the processed tables to `t20_csv_files/`.
6. **Metric computation.** Aggregate fact data per player to derive the metrics used in the role criteria, including batting average, strike rate, innings batted, boundary percentage, average balls faced, batting position, innings bowled, bowling economy, bowling strike rate, bowling average, and dot ball percentage.
7. **Selection.** Apply role-based conditional logic to produce the Best 11.
8. **Visualization.** Generate charts to support and communicate the selection.

## Best 11 Selection Methodology

The Best 11 team is produced by a deterministic, rule-driven process implemented in the notebook. A player qualifies for a role only if every criterion for that role is satisfied. Criteria are strict inequalities, as defined in the Project Sportan scoping document.

### Selection Principles

- Exactly 11 players are selected, each counted once.
- Players are evaluated against the criteria of the role they are being considered for.
- Minimum innings thresholds prevent small sample sizes from distorting results.
- The selected team is validated against the two team-level targets: an average score of at least 180 runs and an average defended total of 150 runs.

### Role Criteria

#### Openers

| Parameter | Description | Criteria |
|-----------|-------------|----------|
| Batting Average | Average runs scored in an innings | > 30 |
| Strike Rate | Runs scored per 100 balls | > 140 |
| Innings Batted | Total innings batted | > 3 |
| Boundary % | Percentage of runs scored in boundaries | > 50 |
| Batting Position | Order in which the batter played | < 4 |

#### Anchors / Middle Order

| Parameter | Description | Criteria |
|-----------|-------------|----------|
| Batting Average | Average runs scored in an innings | > 40 |
| Strike Rate | Runs scored per 100 balls | > 125 |
| Innings Batted | Total innings batted | > 3 |
| Avg. Balls Faced | Average balls faced by the batter in an innings | > 20 |
| Batting Position | Order in which the batter played | > 2 |

#### Finisher / Lower Order Anchor

| Parameter | Description | Criteria |
|-----------|-------------|----------|
| Batting Average | Average runs scored in an innings | > 25 |
| Strike Rate | Runs scored per 100 balls | > 130 |
| Innings Batted | Total innings batted | > 3 |
| Avg. Balls Faced | Average balls faced by the batter in an innings | > 12 |
| Batting Position | Order in which the batter played | > 4 |
| Innings Bowled | Total innings bowled by the player | > 1 |

#### All-Rounders / Lower Order

| Parameter | Description | Criteria |
|-----------|-------------|----------|
| Batting Average | Average runs scored in an innings | > 15 |
| Strike Rate | Runs scored per 100 balls | > 140 |
| Innings Batted | Total innings batted | > 2 |
| Batting Position | Order in which the batter played | > 4 |
| Innings Bowled | Total innings bowled | > 2 |
| Bowling Economy | Average runs allowed per over | < 7 |
| Bowling Strike Rate | Average number of balls required to take a wicket | < 20 |

#### Specialist Fast Bowlers

| Parameter | Description | Criteria |
|-----------|-------------|----------|
| Innings Bowled | Total innings bowled | > 4 |
| Bowling Economy | Average runs allowed per over | < 7 |
| Bowling Strike Rate | Average number of balls required to take a wicket | < 16 |
| Bowling Style | Bowling style of the player | Matches pattern `%Fast%` |
| Bowling Average | Runs allowed per wicket | < 20 |
| Dot Ball % | Percentage of dot balls bowled | > 40 |

### Conditional Logic Flow

1. Compute per-player aggregates from the batting and bowling fact tables and join them to the player dimension.
2. For each role, filter players using all of that role's criteria.
3. Rank the qualifying players within each role.
4. Assemble the team of 11 from the qualifying pools, ensuring no player is selected for more than one role.
5. Validate the team against the average-score target of 180 runs and the average-defended target of 150 runs.

## Visualization Layer

All visualizations are generated in Python:

- `matplotlib` for foundational static plots.
- `seaborn` for statistical and comparative charts.
- `plotly` for interactive exploration.

Typical outputs include distributions of runs and wickets, role-wise performance comparisons, and a summary view of the selected Best 11.

## Assumptions and Limitations

- Results depend on the completeness and accuracy of the scraped source data.
- Selection outcomes are sensitive to the fixed thresholds in the role criteria.
- Statistical measures do not capture match context, pitch conditions, or opposition strength unless explicitly modeled.
- Source website structure changes may require updates to the scraping configuration.

## Future Enhancements

- Add opposition-adjusted and venue-adjusted performance metrics.
- Parameterize selection rules through a configuration file.
- Package the transformation steps as a reusable Python module with automated tests.
- Add data validation checks and schema enforcement for each pipeline stage.
- Support additional tournaments and formats.

## Author

Atharva Lambde

- GitHub: [atharvagit22](https://github.com/atharvagit22)
- Email: atharvalambde@gmail.com
