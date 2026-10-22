# Snowflake Cortex add-on worksheets

**22 worksheets:** one for each of the 21 industry dbt tracks plus NewsAPI. Each works after either the Snowflake Wizard or manual dbt edition; separate Cortex editions are unnecessary because both produce the same mart contract.

Choose a modular delivery path: **basic Fivetran lab → optional dbt Platform add-on → optional Snowflake Cortex add-on**. Cortex requires the completed Snowflake dbt mart and provisioned Snowsight/Cortex access. There is no BigQuery AI equivalent in this series.

Each worksheet includes a complete native semantic-view YAML definition, a Cortex Analyst tool and draft-agent setup, track-specific questions, instructor-led SQL comparisons, and an exercise that checks the assistant's handling of unsupported questions. Direct data previews remain instructor-led.

| Track | Cortex worksheet | Transformed input |
|---|---|---|
| Agriculture | [Worksheet](agriculture_snowflake_cortex_hands_on_lab.md) | `mart_agriculture_review_summary` |
| Attractions | [Worksheet](attractions_snowflake_cortex_hands_on_lab.md) | `mart_attractions_review_summary` |
| Construction | [Worksheet](construction_snowflake_cortex_hands_on_lab.md) | `mart_construction_review_summary` |
| Consumer Packaged Goods | [Worksheet](consumer_packaged_goods_snowflake_cortex_hands_on_lab.md) | `mart_consumer_packaged_goods_review_summary` |
| Customer Service | [Worksheet](customer_service_snowflake_cortex_hands_on_lab.md) | `mart_customer_service_review_summary` |
| Energy | [Worksheet](energy_snowflake_cortex_hands_on_lab.md) | `mart_energy_review_summary` |
| Financial Services | [Worksheet](financial_services_snowflake_cortex_hands_on_lab.md) | `mart_financial_services_review_summary` |
| Healthcare | [Worksheet](healthcare_snowflake_cortex_hands_on_lab.md) | `mart_healthcare_review_summary` |
| Higher Education | [Worksheet](higher_education_snowflake_cortex_hands_on_lab.md) | `mart_higher_education_review_summary` |
| Hospitality | [Worksheet](hospitality_snowflake_cortex_hands_on_lab.md) | `mart_hospitality_review_summary` |
| Insurance | [Worksheet](insurance_snowflake_cortex_hands_on_lab.md) | `mart_insurance_review_summary` |
| Manufacturing | [Worksheet](manufacturing_snowflake_cortex_hands_on_lab.md) | `mart_manufacturing_review_summary` |
| Media Entertainment | [Worksheet](media_entertainment_snowflake_cortex_hands_on_lab.md) | `mart_media_entertainment_review_summary` |
| Pharma | [Worksheet](pharma_snowflake_cortex_hands_on_lab.md) | `mart_pharma_review_summary` |
| Retail | [Worksheet](retail_snowflake_cortex_hands_on_lab.md) | `mart_retail_inventory_priorities` |
| Sales | [Worksheet](sales_snowflake_cortex_hands_on_lab.md) | `mart_sales_review_summary` |
| Supply Chain | [Worksheet](supply_chain_snowflake_cortex_hands_on_lab.md) | `mart_supply_chain_review_summary` |
| Telco | [Worksheet](telco_snowflake_cortex_hands_on_lab.md) | `mart_telco_review_summary` |
| Transportation | [Worksheet](transportation_snowflake_cortex_hands_on_lab.md) | `mart_transportation_review_summary` |
| Utilities | [Worksheet](utilities_snowflake_cortex_hands_on_lab.md) | `mart_utilities_review_summary` |
| Weather | [Worksheet](weather_snowflake_cortex_hands_on_lab.md) | `mart_weather_review_summary` |
| Newsapi | [Worksheet](newsapi_snowflake_cortex_hands_on_lab.md) | `mart_newsapi_source_coverage` |

## Delivery and validation

Start with the [dbt add-on index](../dbt/README.md). Each Cortex worksheet links its two compatible Snowflake prerequisites and its template repository. Restore the dbt baseline before starting; point the semantic view at the development mart and create the semantic view and agent in the attendee's provisioned APPS schema.

Grouped summaries sum stored counts and calculate rates from summed numerators and denominators. Means are reconstructed using measured counts, preserving the underlying observation weighting. Retail uses latest product/store observations; NewsAPI uses deduplicated article coverage, including missing source and title handling.

Validation details are in [validation.md](validation.md). Static and local SQL checks do not establish that Snowflake publication, privileges, or agent responses work in a provisioned account. Run the worksheet's live checks before delivery.
