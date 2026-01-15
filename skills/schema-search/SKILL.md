# Schema Search Skill

This skill helps you search and understand database schemas that have been distilled into lean index files.

## When to Use

Use this skill when the user asks:
- "What tables handle [domain]?"
- "Where is [data] stored?"
- "What columns are in [table]?"
- "Find tables related to [concept]"
- "Schema for [feature]"

## How to Search

1. **Check for distilled schema**: Look for `schema-index.md` in the project root or `.claude/` directory
2. **Use Grep**: Search the index file with relevant keywords
3. **Return matches**: Show table name, purpose, and key columns

## Schema Index Format

Distilled schemas follow this format:

```
## [Domain Name]
table_name | Purpose description | key_column_1, key_column_2, foreign_key_id
```

Example:
```
## Users & Auth
users | Core user accounts | id, email, role, organization_id
user_sessions | Active login sessions | id, user_id, token, expires_at
```

## Search Strategy

1. **By domain**: Search section headers (`## Users`, `## Projects`)
2. **By table name**: Direct grep for table name
3. **By column**: Search for column names to find which tables contain them
4. **By relationship**: Search for `_id` suffixes to find foreign keys

## Response Format

When you find matches, respond with:

```
Found in schema-index.md:

**[Domain]**
- `table_name` - Purpose
  - Key columns: col1, col2, col3
  - Relationships: links to other_table via foreign_key_id
```

## If No Index Found

If no `schema-index.md` exists, tell the user:

"No distilled schema found. Run `/distillery:distill` to generate one from your Supabase database."
