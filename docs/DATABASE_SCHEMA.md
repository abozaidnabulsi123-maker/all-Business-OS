# AI Business OS — Database Schema

## 1. Verified project
The active database containing the current application schema is the Supabase project named `abozaidnabulsi123-maker's Project` (region: `eu-north-1`). The separate project named `ai business os` (region: `eu-west-1`) was inspected and currently returned no public tables and no migrations. Do not confuse the two projects.

The verified project currently has 21 public tables. RLS is enabled on all 21.

## 2. Verified public tables
### Organization and identity
- `profiles`
- `companies`
- `company_members`
- `branches`

### Parties and operations
- `customers`
- `suppliers`
- `products`
- `warehouses`

### Sales and purchasing
- `sales_invoices`
- `sales_invoice_items`
- `sales_receipts`
- `purchase_bills`
- `purchase_bill_items`
- `supplier_payments`

### Inventory and accounting
- `inventory_movements`
- `chart_of_accounts`
- `journal_entries`
- `journal_entry_lines`
- `company_accounting_settings`
- `company_account_mappings`
- `accounting_account_templates`

## 3. Verified organization schema
- `companies`: id, name, legal_name, country, city, phone, email, created_by, created_at, updated_at.
- `branches`: id, company_id, name, address, city, phone, is_main, created_at.
- `company_members`: id, company_id, user_id, role, created_at. Role check permits owner, admin, manager, accountant, employee, viewer.
- `profiles`: id, full_name, phone, created_at, updated_at; id references auth.users.

The company country default is Jordan. The branch and company relationships are enforced with foreign keys.

## 4. Verified accounting and workflow details
- `company_accounting_settings` stores base currency (default JOD), fiscal year start month, and accounting basis (accrual/cash).
- `chart_of_accounts` stores company-specific accounts and account types: asset, liability, equity, revenue, expense.
- `company_account_mappings` maps business purposes such as cash, bank, receivables, inventory, payables, sales revenue, and expenses to company accounts.
- `inventory_movements` records movement type, warehouse, product, quantity in/out, unit cost, reference, and movement date.
- Sales and purchase documents have header and item tables.
- Receipts and supplier payments have triggers that apply payments to their source documents and create accounting postings.

## 5. Verified migrations
The database migration history currently includes:
- `20261111204811_add_warehouse_to_sales_and_purchase_documents`
- `20261111205054_create_atomic_sales_invoice_confirmation`
- `20261111205235_protect_confirmed_sales_invoice_details`
- `20261111205659_create_atomic_purchase_bill_receiving`

## 6. Verified functions and triggers
Functions observed include:
- `create_company`
- `register_company`
- `handle_new_user`
- `is_company_member`
- `has_company_role`
- `confirm_sales_invoice`
- `receive_purchase_bill`
- `auto_post_accounting_source`
- `apply_payment_to_source_document`
- `initialize_company_chart_of_accounts`

Triggers observed protect posted journal entries and document details, automatically post sales invoices and purchase bills after status changes, and apply/post sales receipts and supplier payments.

## 7. Remaining schema work
The current schema is a substantial working foundation, not a complete ERP. Areas still to assess include dedicated product categories and units, stock transfers and controlled adjustments, customer/supplier credit notes and returns, payment allocation edge cases, audit logs, notifications, document storage, and AI activity logs.

## 8. Migration discipline
Before any schema change:
1. Inspect the live schema, policies, triggers, functions, grants, and migration history.
2. Write a focused migration with safe constraints and a rollback plan where possible.
3. Add authorization and transaction tests with the migration.
4. Verify tenant isolation across at least two companies.
5. Update this document only after the migration is confirmed in the target project.

A proposed table or feature is not considered deployed until verified in the live Supabase project.
