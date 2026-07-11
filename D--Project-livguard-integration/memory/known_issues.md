---
name: known-issues
description: Bugs encountered during setup and how they were resolved
metadata: 
  node_type: memory
  type: project
  originSessionId: 2012a5ec-4bf5-4f4a-b028-5e64fa866a9d
---

**LocalDate ClassCastException on write** — Snowflake JDBC throws `ClassCastException: LocalDate cannot be cast to java.sql.Date` when Spring passes LocalDate via setObject(). Fixed by SqlParameterSources.forBean() which converts LocalDate → java.sql.Date before binding. See [[tech-decisions]].

**Boolean isActive Jackson serialization** — Lombok generates isActive() getter for Boolean isActive field. Java beans introspection strips `is` prefix → Jackson serializes as `"active"` not `"is_active"`. Fixed with @JsonProperty("is_active") on the DTO field.

**Boolean isActive BeanPropertySqlParameterSource** — Same Lombok/beans issue: BeanPropertySqlParameterSource sees property `active` not `isActive`. In MERGE SQL for TerritoryMapping, the parameter is `:active` (not `:isActive`) while the column alias remains `AS IS_ACTIVE`.

**Bulk validation not working** — List<@Valid T> on controller method params requires @Validated on the controller class to cascade. Without it, validation on bulk endpoint items is silently skipped.

**Arrow JVM module error** — Snowflake JDBC uses Apache Arrow internally. Requires --add-opens=java.base/java.nio=ALL-UNNAMED in JVM args. Set in build.gradle.kts bootRun block AND IntelliJ Run Configuration VM options. JDBC_QUERY_RESULT_FORMAT=JSON also set to bypass Arrow entirely.

**Snowflake account identifier** — Format is `{locator}.{region}.{cloud}` from the browser URL. Example: mz90275.ap-southeast-7.aws from https://app.snowflake.com/ap-southeast-7.aws/mz90275/

**SecurityConfig ObjectMapper compile error** — `spring-boot-starter-security` addition caused `com.fasterxml.jackson.databind does not exist` in SecurityConfig even though Jackson is a transitive dep of starter-web. Fixed by removing ObjectMapper from the security layer entirely — the 401 response is written as a raw JSON string in ApiKeyAuthFilter instead of using ObjectMapper.
