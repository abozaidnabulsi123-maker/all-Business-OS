# AI Business OS — Database Schema

## 1. Scope and accuracy
This document separates the setup baseline previously observed in the Supabase project from the target design. The live database must be inspected before any migration is applied. A table or field listed under the target design is not considered deployed merely because it appears here.

## 2. Known setup baseline
The following objects were observed during the earlier Supabase setup and should be re-verified against the live project before modification.

### Tables
- `companies`
  - `id`
  - `name`
  - `legal_name`
  - `country` (default recorded as Jordan)
  - `city`
  - `phone`
  - `email`
  - `created_by`
  - `created_at`
  - `updated_at`
- `branches`
  - Includes `company_id`, `name`, `address`, `city`, `phone`, `is_main`, `created_at`.
- `company_members`
  - Company membership and role assignments.
  - Observed role values: `owner`, `admin`, `manager`, `accountant`, `employee`, `viewer`.
- `profiles`
  - User profile records keyed to the authentication user.

### Security and database objects previously observed
- RLS was enabled on `branches`, `companies`, `company_members`, and `profiles`.
- Policies previously observed included:
  - branches: insert, select, update
  - companies: select, update
  - company_members: select
  - profiles: select, update
- Functions previously observed:
  - `create_company`
  - `handle_new_user`
  - `has_company_role`
  - `is_company_member`
  - `register_company`
- An `auth.users` after-insert trigger named for `handle_new_user` was observed creating a profile from user metadata.
- `register_company` was previously configured as a security-definer function for authenticated execution.

## 3. Target domain model
The following is a design target, not a claim of deployed tables.

### Organization and access
- `companies`: tenant root.
- `branches`: company locations.
- `company_members`: user-to-company membership and role.
- `profiles`: user-facing profile details.

### Parties and catalog
- `customers`, `suppliers`, `contacts`
- `product_categories`, `products`, `units`
- `warehouses`

### Sales and purchasing
- `sales_quotations`, `sales_orders`, `sales_invoices`, `sales_invoice_items`
- `sales_returns`, `sales_return_items`, `receipts`
- `purchase_requests`, `purchase_orders`, `purchase_invoices`, `purchase_invoice_items`
- `purchase_returns`, `purchase_return_items`, `payments`

### Inventory and finance
- `stock_movements`, `stock_transfers`, `stock_adjustments`
- `cash_accounts`, `bank_accounts`, `journal_entries`, `journal_entry_lines`
- `accounting_periods`, `payment_allocations`, `receipt_allocations`

### Governance and intelligence
- `audit_logs`, `notifications`, `documents`, `ai_activity_logs`

## 4. Common column conventions
For tenant-owned tables, use as applicable:
- `id uuid primary key`
- `company_id uuid not null references companies(id)`
- `branch_id uuid references branches(id)` when branch scope applies
- `created_at timestamptz not null default now()`
- `updated_at timestamptz not null default now()`
- `created_by uuid` referencing the authenticated user where appropriate

Use explicit foreign keys, unique constraints, check constraints, and indexes. Avoid storing derived balances as the only source of truth.

## 5. Document and ledger integrity
- Invoice headers and line items must be linked and posted atomically.
- Posted documents should not be silently deleted or rewritten.
- Returns and reversals must reference the original document.
- Every stock movement must identify its source document and warehouse.
- Financial journal entries must balance before posting.
- Monetary columns should use fixed-precision numeric types and explicit currency codes.

## 6. Migration checklist
Before implementation:
1. Inspect the live schema, columns, constraints, indexes, triggers, functions, and RLS policies.
2. Compare live state with this baseline.
3. Write a forward migration with a clear rollback strategy where possible.
4. Add RLS policies and role tests in the same delivery.
5. Test with multiple companies to prove tenant isolation.
6. Record the migration and update this document after successful deployment.
