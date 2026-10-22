# Snowflake Cortex worksheet validation

Checked on 2026-10-22. Scope: 22 worksheets, one per existing dbt add-on track, using the completed Snowflake model answers from the matching manual worksheets.

## Checks completed

- Parsed all 22 embedded semantic-view YAML definitions and confirmed each physical table matches its dbt mart.
- Built the completed Snowflake fact and mart SQL locally by rendering dbt references and transpiling Snowflake SQL to DuckDB. The 21 industry tracks used LabStudios seed fixtures; NewsAPI used synthetic URL, source, publication and missing-title fixtures.
- Bound every semantic dimension, time dimension and fact expression against the resulting mart columns, and executed every semantic metric expression. This catches missing or renamed fields, including retail's computed lead-time gap.
- Executed all 44 logical reference queries and their 44 instructor SQL counterparts; each pair produced the same result and ordering.
- Compared reconstructed summary means against direct fact-table means for all 20 grouped-summary tracks, using numeric tolerance. Confirmed a zero measured-count denominator returns null.
- Checked all worksheet-relative links and the 22-track index. The energy prompt explicitly handles the single WTI crude group.
- Reviewed the workflow against Part 8 of the existing retail worksheet and the official [semantic-view YAML specification](https://docs.snowflake.com/en/user-guide/views-semantic/semantic-view-yaml-spec) and [Cortex Agent setup documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-agents-manage).

## What remains a live delivery check

Local SQL execution is not a Snowflake service integration test. No live semantic views or agents were created, no account grants were changed, and no Cortex chat responses were validated. Before delivering a session, use the worksheet to confirm publishing, role/warehouse access, Analyst execution traces, SQL-result agreement and the agent's response to an unsupported question in the provisioned account.

The YAML `verified_queries` entries supply reference examples. They do not claim live verification in an attendee account. Expected counts depend on that account's synced data and restored dbt baseline. NewsAPI fixture checks do not call the external API or establish current news coverage.
