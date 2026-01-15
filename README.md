<p align="center">
  <img src="assets/banner.jpg" alt="Distillery" width="400">
</p>

# Distillery

Stop burning 25k tokens every time Claude needs to check a table name.

## The Problem

You ask Claude to write a query. Claude calls `list_tables` on Supabase MCP. Your context window screams. 25,000 tokens gone. For a table name.

## The Fix

Distill your schema once into a ~300 line grep-friendly file. Claude searches that instead.

```
Before: "What tables handle risks?" → 25k token MCP call
After:  "What tables handle risks?" → grep, 0 tokens, instant
```

## Install

```
/plugin marketplace add Snack-JPG/Distillery
/plugin install distillery@Snack-JPG-Distillery
```

Restart Claude Code.

## Use

**Generate your schema index:**
```
/distillery:distill
```

This creates `schema-index.md` in your project and updates CLAUDE.md so future sessions know about it.

**Then just ask naturally:**
- "What tables handle users?"
- "What columns are in project_risks?"
- "Find tables with organization_id"

The skill intercepts MCP calls and searches your index instead.

## What You Get

A file like this:

```markdown
# Schema Index

## Users & Auth
users | Core accounts | id, email, role, organization_id
user_sessions | Active sessions | id, user_id, token_hash, expires_at

## Projects
projects | Main entity | id, name, status, owner_id
project_risks | Risk register | id, project_id, title, impact_score
```

One line per table. Grep it. Done.

## It Stays Fresh

The skill tells Claude to update `schema-index.md` after schema changes. For automatic reminders, add this hook to your project's `.claude/settings.local.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "mcp__supabase__apply_migration",
        "hooks": [
          {
            "type": "command",
            "command": "echo '📝 Schema changed - update schema-index.md with the table/columns you just created or modified.'"
          }
        ]
      }
    ]
  }
}
```

When Claude runs a migration, it gets reminded to add the new table directly - no need to re-query all 318 tables. Just one line edit.

```bash
# Quick test after install
grep "user" schema-index.md
```

Shows the value prop immediately.

## Real World Test

Ran this on PulseAI - a 543k LOC enterprise SaaS with 318 database tables:

| Metric | Before | After |
|--------|--------|-------|
| Tables | 318 | 318 |
| Schema lookup tokens | ~32,000 | 0 |
| Output file | - | 490 lines |
| Domains auto-organized | - | 40+ |

The AI grouped tables into logical domains: Organizations, Projects, RAID, AI Insights, Pulse AI System, Financial, Resources, and 35 more. One line per table, grep-searchable, foreign keys visible.

## Why

| Approach | Tokens | Speed |
|----------|--------|-------|
| Raw MCP `list_tables` | ~25,000 | Slow |
| **Distilled index** | ~0 | Instant |

---

Built by [Austin](https://github.com/Snack-JPG) because context windows aren't infinite and MCP calls aren't free.
