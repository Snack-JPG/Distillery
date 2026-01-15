---
description: Process a batch of database tables and format them into lean schema index format. Use this agent with model "haiku" for cost efficiency.
tools:
  - mcp
---

# Schema Batch Processor

You are processing a batch of database tables to create a lean, searchable schema index.

## Your Task

1. **Query the tables** assigned to you using Supabase MCP's `execute_sql`:

```sql
SELECT
  t.table_name,
  string_agg(c.column_name, ', ' ORDER BY c.ordinal_position) as columns
FROM information_schema.tables t
JOIN information_schema.columns c
  ON t.table_name = c.table_name AND t.table_schema = c.table_schema
WHERE t.table_schema = 'public'
  AND t.table_type = 'BASE TABLE'
  AND t.table_name IN ($TABLE_LIST)
GROUP BY t.table_name
ORDER BY t.table_name;
```

2. **Format each table** as a single line:
```
table_name | Brief purpose | key_col_1, key_col_2, foreign_key_id
```

## Formatting Rules

- **One line per table**, max 100 characters
- **Infer purpose** from table name and columns (e.g., "project_risks" → "Risk register")
- **Prioritize columns**: id first, then name/title/key fields, then foreign keys (*_id), skip timestamps
- **Max 6-8 columns** per line - pick the most important ones
- **Foreign keys**: Always include columns ending in `_id` as they show relationships

## Example Output

```
users | Core user accounts | id, email, full_name, role, organization_id
project_risks | Risk register entries | id, project_id, title, impact_score, status
audit_log | Change tracking | id, entity_type, entity_id, action, actor_id
```

## Return Format

Return ONLY the formatted lines, one per table. No headers, no markdown, just the pipe-delimited lines.

$ARGUMENTS
