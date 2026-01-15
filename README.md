# Distillery

**Distill database schemas into lean, searchable indexes.**

Turn 25,000 tokens of `list_tables` noise into a ~300 line grep-friendly reference.

## The Problem

```
You: "What tables handle user permissions?"
Supabase MCP: *returns 25k tokens of every column, constraint, and index*
Context window: *cries*
```

## The Solution

```
You: "What tables handle user permissions?"
Claude: *greps schema-index.md*
Claude: "Found: user_roles, permissions, role_permissions - here are the key columns..."
```

## Installation

```bash
# Add as marketplace
/plugin marketplace add Snack-JPG/Distillery

# Install the plugin
/plugin install distillery@Snack-JPG/Distillery
```

Or add directly:
```bash
/plugin install Snack-JPG/Distillery
```

## Usage

### Generate Your Schema Index

```bash
/distillery:distill
```

This will:
1. Query your Supabase schema (requires Supabase MCP connected)
2. Organize tables by domain
3. Output `schema-index.md` in your project root

### Search Your Schema

Once generated, just ask naturally:

- "What tables handle authentication?"
- "Where is project data stored?"
- "What columns are in the users table?"
- "Find tables with organization_id"

The `schema-search` skill will grep your index and return concise results.

## Schema Index Format

```markdown
# Schema Index

Generated: 2026-01-15
Tables: 42

---

## Users & Authentication
users | Core user accounts | id, email, role, organization_id
user_sessions | Active sessions | id, user_id, token_hash, expires_at

## Projects
projects | Main entity | id, name, status, owner_id
tasks | Work items | id, project_id, title, assignee_id
```

**One line per table. Grep-friendly. Token-light.**

## Why This Works

| Approach | Tokens | Searchable |
|----------|--------|------------|
| Raw `list_tables` | ~25,000 | ❌ Noise |
| Full schema dump | ~50,000 | ❌ Overkill |
| **Distilled index** | ~2,000 | ✅ Grep it |

## Manual Schema Index

Don't use Supabase? Create `schema-index.md` manually:

```markdown
# Schema Index

## [Domain]
table_name | Purpose | key_col_1, key_col_2, foreign_key_id
```

The `schema-search` skill will find and use it.

## Requirements

- Claude Code 1.0.33+
- Supabase MCP (for auto-generation) or manual creation

## License

MIT

---

Built by [Austin @ GenFlow Systems](https://github.com/Snack-JPG)

*Bringing knowledge from hard-to-find to immediately accessible.*
