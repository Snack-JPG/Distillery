# Schema Index

Generated: 2026-01-15
Tables: 42
Database: example-saas-app

---

## Users & Authentication
users | Core user accounts | id, email, full_name, role, organization_id, created_at
user_profiles | Extended user data | id, user_id, avatar_url, preferences_json
user_sessions | Active sessions | id, user_id, token_hash, ip_address, expires_at
password_reset_tokens | Password recovery | id, user_id, token_hash, expires_at
oauth_connections | SSO providers | id, user_id, provider, provider_user_id

## Organizations & Teams
organizations | Multi-tenant orgs | id, name, slug, plan, owner_id, settings_json
organization_members | Org membership | id, organization_id, user_id, role, invited_by
teams | Sub-groups within orgs | id, organization_id, name, description
team_members | Team membership | id, team_id, user_id, role

## Projects & Tasks
projects | Core project entity | id, organization_id, name, status, owner_id, due_date
project_members | Project access | id, project_id, user_id, permission_level
tasks | Work items | id, project_id, title, status, assignee_id, priority, due_date
task_comments | Discussion on tasks | id, task_id, user_id, content, created_at
task_attachments | Files on tasks | id, task_id, filename, storage_path, uploaded_by

## Documents & Files
documents | Shared documents | id, project_id, title, content_json, version, created_by
document_versions | Version history | id, document_id, version_number, content_json, created_by
file_uploads | Binary storage refs | id, organization_id, filename, mime_type, storage_url, size_bytes

## Notifications & Activity
notifications | User notifications | id, user_id, type, title, read_at, data_json
activity_log | Audit trail | id, organization_id, actor_id, action, entity_type, entity_id, metadata_json
webhooks | External integrations | id, organization_id, url, events, secret_hash, enabled

## Billing & Subscriptions
subscriptions | Stripe subscriptions | id, organization_id, stripe_subscription_id, status, plan_id
invoices | Payment history | id, subscription_id, stripe_invoice_id, amount_cents, status, paid_at
payment_methods | Stored cards | id, organization_id, stripe_payment_method_id, last_four, is_default

## Settings & Config
feature_flags | Rollout controls | id, name, enabled, rollout_percentage, organization_ids
system_settings | Global config | id, key, value_json, updated_at
email_templates | Notification templates | id, slug, subject, body_html, body_text

---

## Quick Reference

**Foreign Key Patterns:**
- `user_id` → users.id
- `organization_id` → organizations.id
- `project_id` → projects.id
- `*_by` (created_by, updated_by) → users.id

**Status Enums:**
- projects.status: draft, active, on_hold, completed, archived
- tasks.status: todo, in_progress, review, done
- subscriptions.status: trialing, active, past_due, canceled

**Common Queries:**
- "Find user's projects": projects WHERE owner_id = ? OR project_members.user_id = ?
- "Org activity": activity_log WHERE organization_id = ? ORDER BY created_at DESC
- "Overdue tasks": tasks WHERE due_date < NOW() AND status != 'done'
