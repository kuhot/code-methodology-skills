# CLAUDE.md —  Coding Skills for Coding Users

> This file provides guidance to Claude Code and other coding agents when working with code in this repository.

Priority: **Safety > Correctness > Verifiability > Minimal change > Speed**

---

## English Version

### 0. General Principles

- If unsure, ask. Do not guess.
- Do not make hidden assumptions for the user. Surface options, risks, and trade-offs.
- Any production, database, or irreversible operation requires explicit confirmation.
- **Before deleting, overwriting, or irreversibly updating database data, back it up and prepare a rollback plan.**
- When done, report: what changed, how it was verified, the result, and remaining risks.

Priority: **Safety > Correctness > Verifiability > Minimal change > Speed**

---

### Skill 1 — Think Before Coding

**Trigger:** Ambiguous requirements, multiple interpretations, high-impact changes, data or production involvement.

Before coding:

- State your assumptions explicitly. If unsure, ask.
- If multiple interpretations exist, present the options. Do not silently choose one.
- If a simpler approach exists, say so. Push back when needed.
- If something is unclear, stop and identify exactly what is unclear.
- For database, migration, deletion, or production changes, explain risks and rollback first.

Suggested output:

```text
Assumptions:
Options:
Recommendation:
Needs confirmation:
```

---

### Skill 2 — Simplicity First

**Trigger:** New features, refactoring, abstraction design, adding configuration.

Use the minimum code that solves the problem. Do not write speculative code.

- Do not build features the user did not request.
- Do not create abstractions for one-off code.
- Do not add unrequested flexibility or configurability.
- Do not write error handling for impossible scenarios.
- If you wrote 200 lines and 50 would do, rewrite it.

Ask yourself: “Would a senior engineer call this overcomplicated?” If yes, simplify.

---

### Skill 3 — Surgical Changes

**Trigger:** Modifying existing code, bug fixes, small iterations.

Touch only what must be touched. Clean up only your own mess.

When modifying existing code:

- Do not “improve” adjacent code, comments, or formatting.
- Do not refactor things that are not broken.
- Follow the existing style, even if you would write it differently.
- If you find unrelated dead code, mention it. Do not delete it.

When your changes create orphaned code:

- Remove imports, variables, and functions that are no longer used **because of your changes**.
- Do not delete pre-existing dead code unless the user explicitly asks.

Standard: every changed line in the diff must trace directly to the user’s request.

---

### Skill 4 — Goal-Driven Execution

**Trigger:** Multi-step tasks, bug fixes, validation, refactoring, database changes.

Define success criteria first, then iterate until verification passes.

Rewrite tasks as verifiable goals:

- “Add validation” → “Write tests for invalid input first, then make them pass.”
- “Fix this bug” → “Write a reproducing test first, then make it pass.”
- “Refactor X” → “Keep all tests passing before and after.”
- “Change the database” → “Migration runs, rolls back, tests pass, backup verified.”

For multi-step tasks, give a short plan:

```text
1. [Step] → Verify: [Check]
2. [Step] → Verify: [Check]
3. [Step] → Verify: [Check]
```

---

### Skill 5 — Database Safety Rules

**Default: read before write, back up before delete, always rollback, least privilege.**

#### 5.1 Connections, Environments, and Permissions

- Do not hardcode database credentials. Use environment variables or a secret manager.
- Do not commit `.env` files, connection strings, secrets, or tokens.
- Identify the environment explicitly: `local` / `test` / `staging` / `prod`.
- Production database operations require explicit user confirmation.
- Use read-only accounts for read-only tasks. Use least-privilege accounts for writes.

#### 5.2 Queries and Write Operations

- Never run `UPDATE` or `DELETE` without `WHERE`.
- Never run `DROP` or `TRUNCATE` without explicit confirmation.
- Before writes, preview affected rows with `SELECT`; use `LIMIT` when appropriate.
- Batch large changes. Control batch size. Watch locks and latency.
- Put multi-step writes in transactions. Roll back on failure.
- Use migration scripts for DDL. Test them. Provide rollback or forward-fix plans.
- Avoid N+1 queries. Use `EXPLAIN` when performance matters.

#### 5.3 Back Up Before Delete / Update

Before deleting or irreversibly updating data, you must:

1. Confirm the environment and scope.
2. Back up affected data to a timestamped backup table, export file, or snapshot.
3. Verify backup row counts or contents.
4. Provide rollback SQL.
5. Get explicit user confirmation for production.
6. Record affected rows, backup location, and executed statements.

Example:

```sql
CREATE TABLE orders_backup_YYYYMMDD_HHMMSS AS
SELECT * FROM orders WHERE created_at < '2024-01-01';

-- Verify backup
SELECT COUNT(*) FROM orders_backup_YYYYMMDD_HHMMSS;
```

Rules:

- Prefer soft deletes. Physical deletes require confirmation.
- Before cascading deletes, back up child tables.
- If the user asks to skip backups, stop and explain the risk. Unless it is clearly local/test data that can be discarded, still recommend a backup.

#### 5.4 Privacy and Logging

- Do not print or commit PII, secrets, or tokens.
- Redact logs. Use synthetic or anonymized data for tests.

#### 5.5 Pre-Database-Operation Checklist

- [ ] Environment: prod / test?
- [ ] Permission: read-only / write?
- [ ] Backup: backed up and verified?
- [ ] Preview: affected rows selected?
- [ ] Transaction: needed?
- [ ] Rollback: SQL prepared?
- [ ] Confirmation: production delete/update confirmed?
- [ ] Record: affected rows / backup location?

---

### Skill 6 — Verification and Delivery

**Trigger:** Any code or database change is complete.

- Write reproducing tests for bugs; write invalid-input tests for validation.
- Test database changes with up/down migrations or rollback scripts.
- When done, report: assumptions, changes, verification commands, results, risks, and open questions.
- Do not hide uncertainty. Do not call something complete before it is verified.

**Delivery report template:**

```text
Assumptions:
Changes:
Verification commands:
Verification results:
Risks:
Open questions:
```