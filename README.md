# code-methodology-skills

> Skills refined and expanded upon Andrej Karpathy’s four core skills.
> Methodology skills for coding users and AI coding agents.
>
> 🌐 [English](./README.md) · [中文](./README-zh.md)

A collection of reusable behavioral guidelines (Skills) for AI coding agents such as Claude Code, Cursor, and Copilot. The core goals:

- **Safety first** — database and irreversible operations must be backed up and rollback-ready
- **Correctness first** — think before coding, define success criteria before acting
- **Minimal changes** — surgical edits; every diff line traces back to the request
- **Simplicity first** — the least code that solves the problem; no over-engineering

Priority: **Safety > Correctness > Verifiability > Minimal change > Speed**

---

## Table of Contents

- [Repository Structure](#repository-structure)
- [Usage](#usage)
- [Skills Overview](#skills-overview)
- [Database Safety Rules](#database-safety-rules)
- [Delivery Report Template](#delivery-report-template)
- [English Overview](#english-overview)
- [Contributing](#contributing)
- [License](#license)

---

## Repository Structure

```text
code-methodology-skills/
├── README.md          # This file (English)
├── README-zh.md       # Chinese version
├── CLAUDE.md          # Main skills file (bilingual, auto-loaded by Claude Code)
└── LICENSE            # License (optional)
```

- `CLAUDE.md` is the core file. It follows the Claude Code convention and is auto-loaded when placed at the repository root.
- For other tools (Cursor / Copilot / Cline, etc.), copy `CLAUDE.md` into the corresponding rules file:
  - Cursor: `.cursorrules`
  - GitHub Copilot: `.github/copilot-instructions.md`
  - Cline: `.clinerules`
  - Generic: `AGENTS.md`

---

## Usage

### Option 1: Repository-level rules (recommended)

Place a `CLAUDE.md` in your project root:

```bash
# In the target project
curl -o CLAUDE.md https://raw.githubusercontent.com/kuhot/code-methodology-skills/main/CLAUDE.md
```

Or via git submodule:

```bash
git submodule add https://github.com/kuhot/code-methodology-skills.git .skills/code-methodology
ln -s .skills/code-methodology/CLAUDE.md CLAUDE.md
```

### Option 2: User-level global rules

Place it in your home directory to apply to all projects:

```bash
# Claude Code global config
cp CLAUDE.md ~/.claude/CLAUDE.md
```

### Option 3: Prompt template

Paste the relevant Skill section at the start of a conversation for one-off tasks.

---

## Skills Overview

| # | Skill | Trigger | Core Requirement |
|---|-------|---------|------------------|
| 0 | General Principles | All tasks | If unsure, ask. Do not guess. |
| 1 | Think Before Coding | Ambiguous requirements, high-impact changes | Surface assumptions, options, risks |
| 2 | Simplicity First | New features, refactoring, abstraction | Least code; no speculative design |
| 3 | Surgical Changes | Modifying existing code | Touch only what must be touched |
| 4 | Goal-Driven Execution | Multi-step tasks, bug fixes, refactoring | Define success criteria before acting |
| 5 | Database Safety | Any database read/write | Back up first, rollback-ready, least privilege |
| 6 | Verification & Delivery | Task completion | Report changes, verification, risks |

---

## Database Safety Rules

**Default: read before write, back up before delete, always rollback, least privilege.**

### Hard Rules

- ❌ Never run `UPDATE` / `DELETE` without `WHERE`
- ❌ Never run `DROP` / `TRUNCATE` without explicit confirmation
- ❌ Never hardcode database credentials
- ❌ Never commit `.env` files, connection strings, secrets, or tokens
- ✅ **Always back up** before deleting or irreversibly updating data
- ✅ **Production database operations require explicit user confirmation**
- ✅ Preview affected rows with `SELECT` before writes
- ✅ Wrap multi-step writes in transactions; roll back on failure
- ✅ Use migration scripts for DDL; provide up/down

### Backup Example (Before Delete / Update)

```sql
-- 1. Back up affected rows (timestamped table name)
CREATE TABLE orders_backup_YYYYMMDD_HHMMSS AS
SELECT * FROM orders WHERE created_at < '2024-01-01';

-- 2. Verify backup row count
SELECT COUNT(*) FROM orders_backup_YYYYMMDD_HHMMSS;

-- 3. Preview rows to be deleted
SELECT COUNT(*) FROM orders WHERE created_at < '2024-01-01';

-- 4. Execute delete (after confirmation)
DELETE FROM orders WHERE created_at < '2024-01-01';

-- 5. Rollback SQL (if restore is needed)
INSERT INTO orders
SELECT * FROM orders_backup_YYYYMMDD_HHMMSS;
```

### Pre-Database-Operation Checklist

- [ ] Environment: prod / test?
- [ ] Permission: read-only / write?
- [ ] Backup: backed up and verified?
- [ ] Preview: affected rows selected?
- [ ] Transaction: needed?
- [ ] Rollback: SQL prepared?
- [ ] Confirmation: production delete/update confirmed?
- [ ] Record: affected rows / backup location?

---

## Delivery Report Template

After any code or database change, report in this format:

```text
Assumptions:
Changes:
Verification commands:
Verification results:
Risks:
Open questions:
```

---

## English Overview

### Three Non-Negotiables

1. **If unsure, ask.** Do not guess. Do not make hidden assumptions.
2. **Back up before deleting data.** Always prepare a rollback plan.
3. **Production operations require explicit confirmation.** Never act unilaterally.

### Four-Step Workflow

1. **Think before coding.** State assumptions, present options, explain risks.
2. **Write the minimum.** The least code that solves the problem. No speculative design.
3. **Change the minimum.** Every diff line traces back to the user's request.
4. **Verify until it passes.** Define success criteria first, iterate until tests pass.

### Core Priority

**Safety > Correctness > Verifiability > Minimal change > Speed**

---

## Contributing

Issues and PRs are welcome. When adding a new skill:

- Each skill must include: **Trigger + Behavioral Requirements + Output Format (optional)**
- Bilingual (English + Chinese) content is preferred
- Rules involving data, security, or production must follow the **strictest standard**
- Keep it concise; avoid duplicating existing skills

---

## License

MIT License

---

## References

- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [AGENTS.md Convention](https://agents.md/)
- https://github.com/multica-ai/andrej-karpathy-skills