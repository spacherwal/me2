---
name: Spring Boot 3.x upgrade plan
description: Full plan and change log for upgrading from Spring Boot 2.7.2 to 3.5.9 — stashed, pending infra JDK upgrade
type: project
---

## Status (2026-03-23)

Changes are **stashed in IntelliJ** — reverted from dev-optimization branch due to hosted environment still running JDK 17. Must upgrade server JDK to 21 before this can be merged.

**Why Java 21:** HikariCP 6.x (was pinned in build.gradle) requires Java 21. Easiest fix is to remove the explicit HikariCP version and let Spring Boot BOM manage it (gives 5.x which supports Java 17). But team decided to go to Java 21 directly.

---

## build.gradle changes

| What | From | To |
|------|------|----|
| Spring Boot | `2.7.2` | `3.5.9` |
| dependency-management plugin | `1.0.12.RELEASE` | `1.1.7` |
| HikariCP | `6.2.1` (explicit) | remove explicit version (BOM managed) |
| Lombok | `1.18.24` (explicit) | remove explicit version (BOM gives 1.18.38, Java 21 compatible) |
| spring-boot-starter-mail | `2.6.6` (explicit) | remove explicit version |
| spring-boot-devtools | `2.7.4` implementation | `developmentOnly` (no version) |
| junit:junit:4.13.1 | testImplementation | remove (dead JUnit 4 dep) |
| mockito/junit-jupiter versions | explicit `4.8.0`/`5.9.1` | remove (BOM managed) |
| flapdoodle | `de.flapdoodle.embed.mongo:3.5.4` | `de.flapdoodle.embed.mongo.spring3x:4.21.0` |

---

## javax → jakarta renames

| File | Change |
|------|--------|
| `ApiKeyAuthFilter.java` | `javax.servlet.*` → `jakarta.servlet.*` |
| `ExternalScheduler.java` | `javax.annotation.PostConstruct` → `jakarta.annotation.PostConstruct` |
| `EmailService.java` | `javax.mail.MessagingException` → `jakarta.mail.MessagingException` |
| `EmailServiceImpl.java` | `javax.mail.*` → `jakarta.mail.*` |
| `ProcessLeadIdsRequest.java` | `javax.validation.constraints.NotEmpty` → `jakarta.validation.*` |
| `ScheduleConfig.java` | `javax.validation.constraints.*` → `jakarta.validation.constraints.*` |
| `SolarLeadController.java` | `javax.validation.Valid` → `jakarta.validation.Valid` |
| `SchedulingController.java` | `javax.validation.Valid` → `jakarta.validation.Valid` |
| `SnowflakeConfig.java` | `javax.sql.DataSource` → **NO CHANGE** (Java SE, not Jakarta EE) |

---

## Spring Security 6 changes (SecurityConfig.java)

Old chained API → new lambda DSL:
```java
// Before (Spring Security 5.x)
http
    .csrf().disable()
    .formLogin().disable()
    .httpBasic().disable()
    .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
    .and()
    .authorizeRequests()
        .antMatchers("/webhook/**").permitAll()
        .anyRequest().authenticated()
    .and()
    .addFilterBefore(apiKeyAuthFilter, UsernamePasswordAuthenticationFilter.class);

// After (Spring Security 6.x)
http
    .csrf(csrf -> csrf.disable())
    .formLogin(form -> form.disable())
    .httpBasic(basic -> basic.disable())
    .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
    .authorizeHttpRequests(auth -> auth
        .requestMatchers("/webhook/**").permitAll()
        .anyRequest().authenticated()
    )
    .addFilterBefore(apiKeyAuthFilter, UsernamePasswordAuthenticationFilter.class);
```

---

## Test changes

- `@MockBean` → `@MockitoBean` in all 7 controller test files
- Import: `org.springframework.boot.test.mock.mockito.MockBean` → `org.springframework.test.context.bean.override.mockito.MockitoBean`
- Affected: LeadsquareControllerTest, SolarLeadsquareControllerTest, SolarLeadControllerTest, WebhookControllerTest, SchedulingControllerTest, RazorPayControllerTest, JobSchedulingControllerTest

---

## Errors encountered and fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `Unsupported class file major version 65` | `HikariCP:6.2.1` requires Java 21, project was on Java 17 | Upgraded to Java 21 |
| `JCTree$JCImport does not have member field qualid` | Lombok `1.18.24` incompatible with Java 21 | Remove explicit Lombok version, let BOM give 1.18.38 |
| `Could not find de.flapdoodle.embed.mongo.spring30x:4.18.0` | Wrong artifact name and non-existent version | Correct artifact: `de.flapdoodle.embed.mongo.spring3x:4.21.0` |
| `@MockBean deprecated since 3.4.0` | Spring Boot 3.4+ deprecates `@MockBean` | Replace with `@MockitoBean` from `org.springframework.test.context.bean.override.mockito` |

---

## Infra prerequisite

- Hosted environment must be upgraded to **JDK 21** before branch can be deployed
- Changes are stashed locally in IntelliJ on dev-optimization branch
