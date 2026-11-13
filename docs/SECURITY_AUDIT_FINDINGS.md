# Initial Security and Workflow Audit

Inspection date: 2026-11-13

## Scope
Read-only inspection of the active Supabase project `abozaidnabulsi123-maker's Project` (`myicotrbtsnbyutewrhq`). No database schema, policies, data, or permissions were changed during this audit.

## Positive findings
- RLS is enabled on all 21 public tables.
- Company membership helpers `is_company_member` and `has_company_role` derive membership from `auth.uid()`.
- The SECURITY DEFINER functions inspected use an empty search_path and schema-qualified object references.
- `create_company` and `register_company` reject unauthenticated calls, validate required company/branch names, create the company, add the caller as owner, and create a main branch.
- `confirm_sales_invoice` requires an authenticated user, checks a permitted company role, locks the invoice row, requires draft status and an active same-company warehouse, checks invoice lines and product activity, locks stock streams, validates available stock, creates inventory movements, and changes invoice status.
- `receive_purchase_bill` follows a similar authenticated, role-checked, row-locked workflow and creates incoming inventory movements.
- Triggers exist to protect confirmed sales/purchase document details and journal entries, and to apply/post receipt/payment events.

## Findings requiring decisions or tests
### 1. Employee permissions on financial and stock operations
Several INSERT/UPDATE policies include the `employee` role for invoices, purchase bills, receipts, supplier payments, and inventory movements. The confirmation/receiving functions also include employee in their allowed roles. Confirm whether employees should be allowed to confirm/post documents or only prepare drafts.

### 2. Deletion of posted records
DELETE policies are present on several financial and inventory tables for owner/admin. Triggers protect some posted document changes, but the exact delete behavior must be tested. Prefer reversals, returns, and correction documents for posted records. Do not assume a DELETE policy is safe merely because the role is privileged.

### 3. Membership administration
The inspected `company_members` policy list contains SELECT only. Company creation functions create the initial owner membership. A controlled invitation, membership update, and role-change workflow still needs to be designed and tested.

### 4. Branch authorization
Many policies are company-membership based. Verify whether branch restrictions are required for managers and employees and enforce them consistently if so.

### 5. Trigger-function execution grant
The audit observed that `guard_journal_entry_posting` reports EXECUTE available to anon/authenticated in the privilege check. As a trigger function, it is not intended for direct invocation. Review its ACL and keep trigger-only functions minimally exposed.

## Test plan before changing policies
1. Use dedicated test users for all six company roles.
2. Create two isolated test companies.
3. Test SELECT/INSERT/UPDATE/DELETE on each major module as member and non-member.
4. Test draft versus confirmed invoice and purchase bill state transitions.
5. Test insufficient stock, inactive warehouse, inactive product, duplicate confirmation, and concurrent confirmation.
6. Test attempts to modify/delete posted journal entries and inventory movements.
7. Test company membership invitation and role change paths.
8. Record expected and actual outcomes before preparing a migration.

## Conclusion
The project has a meaningful security foundation and transactional business workflows. It is not yet certified production-secure: role intent, destructive permissions, branch scope, and cross-tenant behavior require explicit tests. No permissions were changed as part of this read-only audit.
