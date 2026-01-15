---
description: Generate a lean, searchable schema index from your Supabase database
---

# Distill Schema

Generate a compact, grep-searchable schema index from the connected Supabase database.

## What You'll Do

1. **Query the schema** using Supabase MCP's `list_tables` or direct SQL:
   ```sql
   SELECT
     t.table_name,
     obj_description((t.table_schema || '.' || t.table_name)::regclass) as description,
     string_agg(c.column_name, ', ' ORDER BY c.ordinal_position) as columns
   FROM information_schema.tables t
   JOIN information_schema.columns c
     ON t.table_name = c.table_name AND t.table_schema = c.table_schema
   WHERE t.table_schema = 'public' AND t.table_type = 'BASE TABLE'
   GROUP BY t.table_schema, t.table_name
   ORDER BY t.table_name;
   ```

2. **Organize by domain** - Group related tables together:
   - Users & Auth
   - Core Business Objects
   - Relationships/Junctions
   - Audit/Logging
   - AI/Analytics (if applicable)

3. **Format each table** as:
   ```
   table_name | Brief purpose | key_columns, foreign_keys
   ```

4. **Output to** `schema-index.md` in the project root

## Output Format

```markdown
# Schema Index

Generated: [date]
Tables: [count]
Database: [project-ref or name]

---

## [Domain 1]
table_name | Purpose | id, key_col, other_table_id
another_table | Purpose | id, name, status

## [Domain 2]
...
```

## Guidelines

- **Be concise**: One line per table, max 100 chars
- **Prioritize columns**: id, name/title, status, foreign keys, timestamps last
- **Skip noise**: Don't include every column, just the searchable ones
- **Group logically**: Users should find related tables together
- **Note relationships**: Include `_id` columns to show foreign keys

## After Generation

Tell the user:
1. Schema index saved to `schema-index.md`
2. Token count (should be <5k for most projects)
3. How to search: "Use Grep to search schema-index.md for any table or column"

$ARGUMENTS
