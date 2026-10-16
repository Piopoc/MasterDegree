---
type: course-note
date: 2026-10-09
tags:
  - course
  - ppia
  - security
course: "[[Privacy Preserving Information Access]]"
sources:
  - "[[Intro]]"
ai-first: true
---

## For Future Agent

Study note for the [[Privacy Preserving Information Access]] course covering SQL Injection (SQLi): anatomy, OWASP history, and the PreparedStatement defense. Created 2026-10-09 from the security portion of [[Intro]] (translated from Italian). Key points: what SQLi is and why it ranks high in OWASP Top Ten, string-concatenation vs. precompiled queries, boolean-tautology and destructive-DDL attack patterns.

## 1. What is SQL Injection (SQLi)

SQL Injection is a code-injection vulnerability in which malicious SQL statements are inserted into the input fields of data-driven applications. With it, an attacker can:

- falsify identity,
- tamper with or destroy information,
- repudiate transactions,
- reveal confidential data,
- escalate to server-administrator privileges.

### OWASP Top Ten ranking

| Year | SQLi rank |
|---|---|
| 2013 | **#1** |
| 2017 | **#1** |
| 2021 | **#3** |

## 2. Vulnerable Query vs. PreparedStatement

| Aspect | String concatenation | `PreparedStatement` |
|---|---|---|
| Construction | Query built by joining text; user input can alter the syntactic structure | Query template pre-compiled with `?` placeholders |
| Separation | Code and data mixed | Executable code cleanly separated from entered data |
| Malicious input | Changes the `WHERE` clause / query logic | Treated as a literal parameter - at worst a read error, attack neutralized |

```sql
-- VULNERABLE: input changes query structure
"SELECT * FROM users WHERE name = '" + input + "'"

-- SAFE: placeholder is data, never code
PreparedStatement stmt = conn.prepareStatement(
    "SELECT * FROM users WHERE name = ?");
stmt.setString(1, input);
```

## 3. Anatomy of an Attack: Two Execution Patterns

### Boolean tautology - logic manipulation of `WHERE`

```
0 OR 1=1
```

Always-true condition bypasses authentication and exposes the entire database. The attacker does not need to know any credentials - they rewrite the *semantics* of the query (open question).

### Destructive DDL - "Little Bobby Tables"

```
Robert'); DROP TABLE Students;--
```

Injects a destructive Data Definition Language command; the table is dropped when the query executes. This is the extreme case of code/data confusion (open question).

### Operational consequences by attack type

- **Confidentiality** - tautologies dump whole tables.
- **Integrity** - destructive DDL or `UPDATE`/`DELETE` injections corrupt or erase data.
- **Repudiation** - transactions can be altered so records no longer reflect what happened.
- **Privilege escalation** - injection through admin-only statements can yield full server control.

## 4. Glossary (this note's terms)

| Term | Definition |
|---|---|
| **SQL Injection (SQLi)** | Attack that inserts malicious SQL into application input fields to alter query syntax and run unauthorized commands. |
| **Prepared Statement** | Defense that pre-compiles the query and parameterizes inputs with `?` placeholders, so user text can never modify the executable statement. |

## Related

- [[Database Foundations and Architecture]] - what SQL and queries are
- [[Data Security and Privacy Fundamentals]] - security as prerequisite of privacy

## Sources

- [[Intro]] (AY 2026/2027)
