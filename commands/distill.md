---
description: Generate a lean, searchable schema index from your Supabase database
---

# Distill Schema

Generate a compact, grep-searchable schema index from the connected Supabase database.

## Step 1: Get Table List (Lightweight Query)

First, run this query to get just table names - this is tiny and fits easily:

```sql
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'public' AND table_type = 'BASE TABLE'
ORDER BY table_name;
```

## Step 2: Batch the Tables

Split the table list into batches of **40 tables each**. For example:
- Batch 1: tables starting A-D (or first 40 alphabetically)
- Batch 2: tables starting E-L (or next 40)
- etc.

## Step 3: Parallel Subagent Processing

For each batch, spawn a **Haiku subagent** using the Task tool with `model: "haiku"`.

Each subagent should:
1. Query only its assigned tables:
```sql
SELECT
  t.table_name,
  string_agg(c.column_name, ', ' ORDER BY c.ordinal_position) as columns
FROM information_schema.tables t
JOIN information_schema.columns c
  ON t.table_name = c.table_name AND t.table_schema = c.table_schema
WHERE t.table_schema = 'public'
  AND t.table_type = 'BASE TABLE'
  AND t.table_name IN ('table1', 'table2', ...) -- batch list
GROUP BY t.table_name
ORDER BY t.table_name;
```

2. Format each table as:
```
table_name | Brief purpose (infer from name/columns) | key_columns
```

3. Return the formatted output

**IMPORTANT**: Run all batch subagents in parallel using multiple Task tool calls in a single message.

## Step 4: Combine Results

Collect outputs from all subagents and organize into domains:

```markdown
# Schema Index

Generated: [date]
Tables: [count]
Database: [project name]

---

## Users & Authentication
[tables related to users, auth, sessions, roles]

## Core Business Objects
[main entities - projects, organizations, etc.]

## Relationships & Junctions
[many-to-many tables, *_members, *_assignments]

## Audit & Logging
[activity_log, audit_*, change_history]

## AI & Analytics
[insights, embeddings, canonical_*, pulse_*]

## Settings & Config
[feature_flags, settings, templates]
```

## Step 5: Save Output

Write the combined, organized schema to `schema-index.md` in the project root.

## Formatting Guidelines

- **One line per table**, max 100 chars
- **Prioritize columns**: id, name/title, status, foreign keys (skip timestamps)
- **Note relationships**: Include `_id` columns to show foreign keys
- **Infer purpose**: Use table/column names to write brief descriptions

## Example Output Line

```
project_risks | Risk register entries | id, project_id, title, impact_score, likelihood_score, status
```

## Step 6: Update CLAUDE.md

Add a reference to the schema index in the project's CLAUDE.md file.

If CLAUDE.md exists, append this section (if not already present):

```markdown
## Database Schema Reference

This project has a distilled schema index at `schema-index.md`.

**ALWAYS check `schema-index.md` before using Supabase MCP for schema lookups.**

- Use Grep to search for tables, columns, or relationships
- Update the index after any CREATE/ALTER/DROP operations
- Only fall back to MCP if the index doesn't have what you need

To regenerate: `/distillery:distill`
```

If CLAUDE.md doesn't exist, create it with the above content.

## After Generation

Tell the user:
1. Schema index saved to `schema-index.md`
2. CLAUDE.md updated with schema reference
3. Total tables processed
4. Approximate token count (should be <3k for most projects)

$ARGUMENTS
