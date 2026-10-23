# dbt add-on validation evidence

The original checks below were performed locally on October 22, 2026. The separate
October 23 addendum records the live Agriculture BigQuery walkthrough; it does not
extend live validation to the other industries or the Wizard flow.

## Inputs

- Industry definitions and seed data: LabStudios commit `5213c55c639e140d38abd4e9b01eb8a6a32c676c`, files `seeds/industry-<industry>-v1.sql`. Those files were unchanged in the inspected checkout.
- Template revisions are pinned below. Existing template repositories were read only.
- NewsAPI template: commit `a2a87afa140d44e9b276d00b9c9de9cc0c69ddd4` in [dbt-newsapi-lab](https://github.com/dbt-Hands-on-Labs/dbt-newsapi-lab).
- NewsAPI fixtures are synthetic: duplicate URLs across endpoints, a missing source ID, an unmatched source ID, a null title/publication timestamp, and a directory match. No NewsAPI requests or article exports were used.

## Checks performed

1. Rendered the Jinja in every answer model for Snowflake and BigQuery. Parsed each SQL dialect with SQLGlot, transpiled it to DuckDB and executed it against the selected LabStudios seed tables, preserving their declared SQL types. This checks logic and column contracts; transpilation is not native warehouse validation.
2. Executed the answer's uniqueness, nullability, relationship and singular reconciliation checks. Compared baseline, changed-threshold and restored-threshold outcomes for every industry and both destinations. Each restored result matched its baseline.
3. Checked NewsAPI with Everything disabled and enabled for both destinations. The synthetic inputs produced three and four deduplicated articles respectively, retaining null/unmatched source IDs. The changed threshold altered short-title counts and restoration returned the baseline.
4. Ran `dbt parse --no-partial-parse` on 44 complete template-plus-answer projects with separate Snowflake and BigQuery profiles containing placeholder credentials. No warehouse connection was opened. Also parsed NewsAPI with Everything false and true, verifying the optional source/model disappeared or appeared in the manifest as intended.
5. Checked 88 worksheet files, local Markdown links, balanced code fences, whitespace, and all 220 manual-answer file blocks against the locally validated files. BigQuery worksheets have no destination-console links or attendee console requirements.

Tools used: dbt-core 1.12.6, dbt-snowflake 1.12.4, dbt-bigquery 1.12.1, duckdb 2.0.0, sqlglot 30.24.0.

The local parser is dbt Core, not the provisioned dbt v2 runtime used in the teaching
workflow. Before delivery, validate the selected edition in the actual lab account,
including v2/State behavior, permissions, generated source names, deployment profiles
and feature availability. No cloud jobs, production settings or LabStudios mappings
were changed by these checks.

## Industry results and template revisions

Counts below are fixture observations only, not expected results attendees must match.
Both dialects produced the same aggregates in the local execution. For Retail, flagged
counts apply to the latest product/store mart, while fact rows retain weekly history.
For other industries, flags apply to fact observations. Missing metrics remain unknown.

| Industry | Template revision | Fact rows | Baseline flags | Changed flags | Restored flags |
|---|---|---:|---:|---:|---:|
| agriculture | [58119230](https://github.com/dbt-Hands-on-Labs/dbt-agriculture-lab/commit/58119230528d04a80b5825f4a2a6386ebb74c2a0) | 5,746 | 1,319 | 609 | 1,319 |
| attractions | [504492cc](https://github.com/dbt-Hands-on-Labs/dbt-attractions-lab/commit/504492cc8ff52c9e056b5eece15d9207b4f8c850) | 210 | 13 | 0 | 13 |
| construction | [2f54e66c](https://github.com/dbt-Hands-on-Labs/dbt-construction-lab/commit/2f54e66c97b3846478337eb864008803633b4e65) | 750 | 199 | 55 | 199 |
| consumer_packaged_goods | [936baff6](https://github.com/dbt-Hands-on-Labs/dbt-consumer-packaged-goods-lab/commit/936baff6e5221ddf0946070b5bfe8de0422700f0) | 750 | 22 | 0 | 22 |
| customer_service | [f27f4c0d](https://github.com/dbt-Hands-on-Labs/dbt-customer-service-lab/commit/f27f4c0deec093777b2f0c9e207a1e8860dd71e7) | 26,872 | 44 | 585 | 44 |
| energy | [c2b9a156](https://github.com/dbt-Hands-on-Labs/dbt-energy-lab/commit/c2b9a156a383503f41808c97c5b8b36188c101d7) | 5,898 | 2,160 | 552 | 2,160 |
| financial_services | [f4f159d6](https://github.com/dbt-Hands-on-Labs/dbt-financial-services-lab/commit/f4f159d67675b4b1485369dcc4f5c21af766a18f) | 39,717 | 11,098 | 3,003 | 11,098 |
| healthcare | [9109a86a](https://github.com/dbt-Hands-on-Labs/dbt-healthcare-lab/commit/9109a86a3dd1e1ec993c92c2d31591ebc0a9374d) | 5,266 | 2,517 | 1,962 | 2,517 |
| higher_education | [479ad0a8](https://github.com/dbt-Hands-on-Labs/dbt-higher-education-lab/commit/479ad0a8e6c02b8e7d267414eff11e5f35936f43) | 751 | 247 | 412 | 247 |
| hospitality | [a26b66c2](https://github.com/dbt-Hands-on-Labs/dbt-hospitality-lab/commit/a26b66c2318bee44eabd1c27b433bbbfc62ea176) | 750 | 290 | 14 | 290 |
| insurance | [ff5d93eb](https://github.com/dbt-Hands-on-Labs/dbt-insurance-lab/commit/ff5d93ebc8ecbce44d0b55106cecb31a346a9d61) | 1,000 | 660 | 191 | 660 |
| manufacturing | [e8d8f3de](https://github.com/dbt-Hands-on-Labs/dbt-manufacturing-lab/commit/e8d8f3de632f24eb7d2e50f5fc66dee41cad894d) | 752 | 372 | 0 | 372 |
| media_entertainment | [f07bfb49](https://github.com/dbt-Hands-on-Labs/dbt-media-entertainment-lab/commit/f07bfb495ef0644af7c9c829dfe52c4f0536f19b) | 8,807 | 525 | 1,335 | 525 |
| pharma | [36d0909d](https://github.com/dbt-Hands-on-Labs/dbt-pharma-lab/commit/36d0909d92911227d7e9bf850cb4236fb703dc96) | 750 | 574 | 485 | 574 |
| retail | [bfe544e4](https://github.com/dbt-Hands-on-Labs/dbt-retail-lab/commit/bfe544e4735f7e896be367c50e4a86fa428d0080) | 28,236 | 116 | 198 | 116 |
| sales | [2529466a](https://github.com/dbt-Hands-on-Labs/dbt-sales-lab/commit/2529466a562d33bd617c54fa700f8d813b74970c) | 755 | 0 | 360 | 0 |
| supply_chain | [02d99b73](https://github.com/dbt-Hands-on-Labs/dbt-supply-chain-lab/commit/02d99b730515cd33e02425dc8a9029b939dfeaba) | 750 | 451 | 273 | 451 |
| telco | [74d1e125](https://github.com/dbt-Hands-on-Labs/dbt-telco-lab/commit/74d1e1250305140a29057837bfbbb51f7aa5f956) | 750 | 123 | 36 | 123 |
| transportation | [a18986e0](https://github.com/dbt-Hands-on-Labs/dbt-transportation-lab/commit/a18986e034447be5b69bb2d16b440904be3fc54e) | 101,315 | 45,486 | 13,286 | 45,486 |
| utilities | [16d8dfe0](https://github.com/dbt-Hands-on-Labs/dbt-utilities-lab/commit/16d8dfe09388a9fec89b16c7cff12f22a42ba9a6) | 750 | 349 | 126 | 349 |
| weather | [0dbc22f6](https://github.com/dbt-Hands-on-Labs/dbt-weather-lab/commit/0dbc22f6ca97e64e86526c42baaad913dd38c86b) | 10,966 | 2,850 | 245 | 2,850 |

## October 23: live Agriculture BigQuery walkthrough

The manual Agriculture worksheet was exercised through Parts 2–6 in dbt Studio and
an isolated General deployment environment using v2 Stable (Fusion 2.0.9). The
instructor corrected the shared connection location and approved removal of the
empty staging dataset left by the earlier failed build. Region provisioning remains
an instructor/LabStudios responsibility, outside the participant worksheet flow.

Two corrections were verified live:

- BigQuery rejected scalar reconciliation SELECTs with WHERE but no FROM. Adding
  `from (select 1) as singleton` preserved the checks and passed the singular test.
- The deployment job form rejected the Studio inline `dbt show` command. A supplied
  macro invoked with `dbt run-operation` logged the same totals and effective target
  successfully in Studio and in the deployment job.

| Run | Models built | Models reused | Tests executed | Tests reused | Flagged |
|---|---:|---:|---:|---:|---:|
| State baseline, threshold 150 | 3 | 0 | 13 | 0 | 1,319 |
| Unchanged rerun | 0 | 3 | 0 | 13 | Not separately queried |
| Threshold 200 | 1 | 2 | 11 | 2 | 609 |
| Restored threshold 150 | 1 | 2 | 0 | 13 | 1,319 |
| Isolated deployment | 3 | 0 | 0 | 13 | 1,319 |

Each aggregate check returned 5,746 observations, 5,746 measured and zero missing
metrics. Reused test results were not freshly executed. These counts describe this
lab's input, not constants participants should expect.

The manually triggered deployment succeeded with the verified personal-branch commit.
Its marts and staging relations used the separately assigned deployment datasets.
The job inherited runtime and State, used No deferral, and left source freshness,
scheduled triggers and job-completion triggers off.

The matching corrections were propagated to all 44 BigQuery worksheets, including
manual-answer SQL and Wizard guidance. Each deployment helper uses its worksheet's
own Part 4 query; Retail and NewsAPI retain their distinct aggregate columns. Static
checks verify helper/query agreement, command references, scalar FROM clauses and
Markdown formatting. Other industry helpers and Wizard flows have not been rerun
against live warehouses as part of this propagation.
