# Schema Search Skill

This skill helps you search and understand database schemas efficiently, avoiding expensive MCP calls.

## When to Use This Skill

**ALWAYS check `schema-index.md` FIRST before using Supabase MCP for schema lookups.**

Trigger this skill when:

### User Questions
- "What tables handle [domain]?"
- "Where is [data] stored?"
- "What columns are in [table]?"
- "Find tables related to [concept]"
- "Schema for [feature]"
- "What's the structure of [table]?"

### Before MCP Calls
**INTERCEPT these patterns** - search schema-index.md instead:
- About to call `list_tables` on Supabase MCP
- About to call `execute_sql` with `information_schema` queries
- About to query `pg_catalog` for table metadata
- Need to verify a table exists before writing a query
- Need to find foreign key relationships
- Looking up column names for a table
- Checking what tables reference another table

### When Writing Queries
- Need to reference existing table names
- Need to know column names for SELECT/INSERT/UPDATE
- Need to find junction tables for JOINs
- Need to understand foreign key relationships
- Validating table/column names before executing SQL

## How to Search

1. **Check for schema index**: Look for `schema-index.md` in:
   - Project root
   - `.claude/` directory
   - `docs/` directory

2. **Use Grep**: Search the index with relevant keywords
   ```
   Grep pattern="users" path="schema-index.md"
   Grep pattern="_id" path="schema-index.md"  # Find foreign keys
   Grep pattern="## " path="schema-index.md"  # List all domains
   ```

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

## Search Strategies

**By domain**: Search section headers
```
Grep pattern="## Users" path="schema-index.md"
```

**By table name**: Direct search
```
Grep pattern="project_risks" path="schema-index.md"
```

**By column**: Find tables containing a column
```
Grep pattern="organization_id" path="schema-index.md"
```

**By relationship**: Find foreign keys
```
Grep pattern="user_id" path="schema-index.md"  # Tables linked to users
```

**List all tables**: Get overview
```
Grep pattern="^[a-z]" path="schema-index.md"  # All table lines
```

## Response Format

When you find matches:

```
Found in schema-index.md:

**[Domain]**
- `table_name` - Purpose
  - Key columns: col1, col2, col3
  - Links to: other_table (via foreign_key_id)
```

## When to Fall Back to MCP

Only use Supabase MCP for schema queries when:
1. `schema-index.md` doesn't exist → tell user to run `/distillery:distill`
2. Need real-time data (row counts, index stats)
3. Need constraint details not in the index
4. Schema has changed and index is stale

## If No Index Found

If no `schema-index.md` exists, tell the user:

"No schema index found. Run `/distillery:distill` to generate one - this will make schema lookups much faster and save tokens."

## After Schema Changes

**ALWAYS update `schema-index.md` after modifying the database schema.**

Update the index when you:
- CREATE TABLE → Add new line to appropriate domain section
- ALTER TABLE ADD COLUMN → Update the table's column list
- ALTER TABLE DROP COLUMN → Remove from column list
- DROP TABLE → Remove the line
- RENAME TABLE → Update the table name
- Create a new migration that changes schema

### How to Update

1. **Find the right section** (or create one):
   ```
   Grep pattern="## Users" path="schema-index.md"
   ```

2. **Add/edit the table line**:
   ```
   table_name | Brief purpose | key_columns
   ```

3. **Use Edit tool** to update the file in place

### Example: After CREATE TABLE

```sql
CREATE TABLE project_comments (
  id uuid PRIMARY KEY,
  project_id uuid REFERENCES projects(id),
  user_id uuid REFERENCES users(id),
  content text,
  created_at timestamptz
);
```

Add to schema-index.md under `## Projects`:
```
project_comments | Discussion on projects | id, project_id, user_id, content
```

### Example: After ALTER TABLE

```sql
ALTER TABLE projects ADD COLUMN archived_at timestamptz;
```

Update the projects line to include `archived_at` if it's a key column (skip if it's just a timestamp).

## Priority Order

1. **First**: Search `schema-index.md` (instant, ~0 tokens)
2. **Only if needed**: Fall back to Supabase MCP (slow, ~25k tokens)
3. **After changes**: Update `schema-index.md` to stay in sync
