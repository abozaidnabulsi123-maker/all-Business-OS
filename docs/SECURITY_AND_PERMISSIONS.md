# AI Business OS — Security and Permissions

## 1. Verified security baseline
The inspected Supabase project is `abozaidnabulsi123-maker's Project` (`eu-north-1`). RLS is enabled on all 21 public tables.

The separate Supabase project named `ai business os` was empty at inspection time. Confirm the project ID before applying any future migration.

## 2. Roles
The `company_members.role` constraint permits:
- `owner`
- `admin`
- `manager`
- `accountant`
- `employee`
- `viewer`

## 3. Verified policy model
Most company-owned tables use `is_company_member(company_id)` for SELECT and `has_company_role(company_id, ...)` for write access.

Observed patterns:
- Branches: members can read; owner/admin/manager can insert and update.
- Companies: members can read; owner/admin can update.
- Company memberships: SELECT policy for company members; no direct INSERT/UPDATE/DELETE policy was observed.
- Profiles: users can read their own profile and profiles of users sharing a company; users can update their own profile.
- Customers, suppliers, products, warehouses, sales/purchase documents, receipts, payments, inventory movements, and journal records: members can read; role-based policies permit writes.
- Chart of accounts: members can read; owner/admin/manager/accountant can insert and update; owner/admin can delete.
- Accounting mappings and settings have dedicated company role policies.
- Accounting account templates have an authenticated SELECT policy with a true condition; this appears to be shared reference data.

## 4. Important findings requiring review
These are review items, not confirmed vulnerabilities:
1. The current policies allow employee-level INSERT and UPDATE on several financial and operational tables, including invoices, purchase bills, payment records, and inventory movements. Confirm whether this matches the intended role matrix.
2. DELETE policies exist for inventory movements, sales and purchase documents, receipts, supplier payments, and journal records, generally for owner/admin. Posted ledger records should normally be corrected by reversal/return workflows rather than deleted.
3. `company_members` has only a SELECT policy in the inspected policy list. Membership creation and role changes must be safely handled through trusted functions or a separately secured workflow.
4. Branch-level scoping is not uniformly visible in every policy expression. Confirm whether company-wide membership is sufficient for each role or whether branch restrictions are required.
5. RLS being enabled does not prove the policy logic is correct. Cross-tenant and role-based tests are still required.

## 5. Functions and privileged execution
Functions observed include `create_company`, `register_company`, `handle_new_user`, `is_company_member`, and `has_company_role`. The membership/role helpers and company registration functions were observed as SECURITY DEFINER. Review each function's search_path, object qualification, auth.uid() checks, EXECUTE grants, and behavior under untrusted input.

Other workflow functions include invoice confirmation, purchase bill receiving, accounting auto-posting, and payment application. These require transaction and authorization tests.

## 6. Trigger protections observed
Triggers currently exist to guard journal entry changes, protect sales invoice and purchase bill headers/items, automatically post confirmed sales and purchase documents, and apply/post sales receipts and supplier payments.

The presence of a trigger is not a substitute for testing its exact state-transition and authorization behavior.

## 7. Required role test matrix
Test each relevant operation as:
- unauthenticated user
- authenticated non-member
- viewer
- employee
- accountant
- manager
- admin
- owner

Also create two test companies and prove that membership in one company never grants access to the other. Test both allowed and denied INSERT, SELECT, UPDATE, and DELETE operations.

## 8. Target policy principles
- Enforce company membership and role server-side.
- Enforce branch scope where the business rule requires it.
- Keep posted stock and financial ledgers immutable to ordinary users.
- Use reversal, credit note, return, or adjustment workflows for posted records.
- Restrict role changes and company membership administration to trusted workflows.
- Log sensitive actions with actor, company, target, timestamp, and outcome.
- AI may recommend actions but must not approve payments, post high-impact transactions, delete records, or change permissions without explicit human authorization.

## 9. Release gate
Do not call the security model production-ready until the role matrix, cross-company isolation, privileged functions, destructive operations, and posted-document protections have been tested against the live database.
