# SQL Filtering for Security Log Investigation

**Type:** Query-based investigation | **Tool:** SQL | **Focus:** Filtering login logs and employee records for suspicious activity and access reviews

## Scenario

A SOC analyst rarely reads logs one row at a time — most real investigation starts with a targeted query against a database or SIEM index. This project applies core SQL filtering operators (`WHERE`, `AND`, `OR`, `NOT`, `LIKE`) against a simulated `log_in_attempts` table and an `employees` table to answer specific investigative and access-review questions.

## Queries

### 1. After-hours failed login attempts

```sql
SELECT *
FROM log_in_attempts
WHERE success = FALSE
  AND login_time > '18:00:00';
```

Filters login attempts to only those that both failed (`success = FALSE`) and occurred after 6 PM. Combining a failure condition with an off-hours time window is a standard first pass for spotting brute-force or unauthorized-access attempts outside normal business activity.

### 2. Login attempts on specific dates

```sql
SELECT *
FROM log_in_attempts
WHERE login_date = '2022-05-09'
   OR login_date = '2022-05-08';
```

Uses `OR` to pull every login attempt across two specific dates — the pattern used when an incident window has already been narrowed down and every event in that window needs to be pulled for review.

### 3. Login attempts originating outside Mexico

```sql
SELECT *
FROM log_in_attempts
WHERE country NOT LIKE 'MEX%';
```

`NOT LIKE 'MEX%'` excludes any country value starting with "MEX" (covers both `MEX` and `MEXICO` formatting inconsistencies in the source data) — useful when the organization's expected user base is concentrated in one country and anything outside it warrants a closer look.

### 4. Employees in Marketing, East building

```sql
SELECT *
FROM employees
WHERE department = 'Marketing'
  AND office LIKE 'East%';
```

Combines an exact match (`department`) with a pattern match (`office LIKE 'East%'`) to scope a query to a specific department **and** physical location — relevant when an investigation or access review needs to be limited to one office/building.

### 5. Employees in Finance or Sales

```sql
SELECT *
FROM employees
WHERE department = 'Finance'
   OR department = 'Sales';
```

`OR` pulls records matching either condition — used to scope an access review across two related departments at once (e.g., both handle financial data).

### 6. Employees not in IT

```sql
SELECT *
FROM employees
WHERE department NOT LIKE 'IT';
```

`NOT LIKE` excludes a specific group — useful for a targeted review of "everyone except the group that's expected to have elevated access."

## Why this matters for SOC work

Every one of these patterns maps directly to a real SIEM/database task: `AND` narrows results to events that meet multiple conditions at once (failed + after hours), `OR` broadens a search across a known set of values (two dates, two departments), `NOT LIKE` excludes a baseline so only the exception surfaces, and `LIKE` with a wildcard handles inconsistent data formatting — which real log data almost always has. Knowing these operators cold is what separates "I can read a dashboard" from "I can go get the answer myself" in a Tier 1 investigation.

## Conclusion

These six queries cover the core filtering logic used to triage login activity and scope access reviews: isolating suspicious time windows, pulling data across specific dates, flagging geographic anomalies, and scoping employee records by department and location.

---
*Part of a self-directed cybersecurity training program (Google Cybersecurity Professional Certificate). See the [main portfolio](../README.md) for other projects.*
