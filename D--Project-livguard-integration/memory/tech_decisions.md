---
name: tech-decisions
description: Key architectural decisions made during initial setup
metadata: 
  node_type: memory
  type: project
  originSessionId: 2012a5ec-4bf5-4f4a-b028-5e64fa866a9d
---

**JdbcClient over Spring Data JDBC** — Spring Data JDBC 4.x has no Snowflake dialect. Attempting to use it causes `Couldn't determine Dialect for "snowflake"` and `JdbcAggregateOperations` bean never registers. Use `JdbcClient` (spring-boot-starter-jdbc) with explicit MERGE SQL instead.

**Why:** Spring Data JDBC dialect detection fails at startup for Snowflake.

**JDBC_QUERY_RESULT_FORMAT=JSON in SnowflakeConfig** — Set on HikariCP datasource properties to prevent Snowflake JDBC from using Apache Arrow result format. Arrow requires `--add-opens=java.base/java.nio=ALL-UNNAMED` which caused JVM module errors.

**Snowflake MERGE INTO for upsert** — All repositories use MERGE INTO with a USING (SELECT :param AS COL) subquery pattern. This is the standard Snowflake upsert pattern.

**SqlParameterSources.forBean()** — Wrapper around BeanPropertySqlParameterSource that converts LocalDate → java.sql.Date before binding. Snowflake JDBC 3.16.1 does not accept java.time.LocalDate in setObject().

**Flat table design** — Nested DTOs (ContactDto, AddressDto, GeoDto, PotentialDto) used for API contract. Service layer maps nested ↔ flat. Models are plain POJOs.

**BeanPropertyRowMapper column mapping** — Snowflake returns UPPER_SNAKE_CASE column names. BeanPropertyRowMapper lowercases then converts underscores to camelCase: RETAILER_ID → retailerId. Works without any custom configuration.
