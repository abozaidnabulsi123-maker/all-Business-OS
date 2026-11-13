# AI Business OS — Security and Permissions

## 1. Security goals
- Prevent users from reading or changing another company's data.
- Enforce permissions in the database and application service layer.
- Keep an auditable record of sensitive business actions.
- Make privileged operations explicit and reviewable.

## 2. Roles
The roles recorded in the initial company-membership setup are:

| Role | Intended access |
|---|---|
| owner | Full company administration and ownership-sensitive actions |
| admin | Company configuration and broad operational administration |
| manager | Operational oversight within assigned company/branch scope |
| accountant | Financial documents, receipts, payments, and accounting reports |
| employee | Assigned day-to-day operational tasks |
| viewer | Read-only access to explicitly permitted information |

These are intended permission boundaries. The effective permissions must be implemented and tested in database policies and application services; role names alone do not enforce access.

## 3. Mandatory authorization checks
For every company-owned record:
1. Obtain the user ID from the authenticated session.
2. Confirm that the user is a member of the record's company.
3. Confirm the required role or explicit permission.
4. Enforce branch scope where applicable.
5. Reject access if any check fails.

Never trust a company_id, user_id, role, price, invoice total, or approval flag supplied by an untrusted client without server-side validation.

## 4. Row Level Security
- Enable RLS on every tenant-owned table.
- Use restrictive policies that bind access to authenticated company membership.
- Separate SELECT, INSERT, UPDATE, and DELETE policies where their rules differ.
- For INSERT and UPDATE, validate both the existing row and the proposed row scope.
- Avoid broad policies such as unconditional authenticated-user access.
- Test cross-company reads and writes explicitly.

## 5. Sensitive operations
The following actions require elevated authorization and audit logging:
- Creating or disabling users and changing roles.
- Changing company or branch settings.
- Posting or reversing invoices and journal entries.
- Recording or approving payments and receipts.
- Posting stock adjustments and transfers.
- Exporting sensitive company data.
- Deleting or archiving business records.

AI assistance must not independently approve payments, post high-impact transactions, delete records, or change access rights. Require a human approval step for configured high-impact actions.

## 6. Database functions
Functions using SECURITY DEFINER must:
- Set a safe, explicit search_path.
- Fully qualify referenced objects where appropriate.
- Validate auth.uid() and membership/role inside the function.
- Grant EXECUTE only to the minimum required roles.
- Avoid exposing privileged behavior to anon or PUBLIC.
- Be reviewed whenever their owner or grants change.

## 7. Audit events
For sensitive operations, record at least:
- company_id
- actor_user_id
- action
- target_table and target_id
- timestamp
- outcome
- relevant before/after metadata, excluding secrets

Audit records should be protected from ordinary user modification.

## 8. Minimum authorization test matrix
Test each operation as:
- unauthenticated user
- authenticated non-member
- viewer
- employee
- accountant
- manager
- admin
- owner

Also test two separate companies with overlapping roles to prove that membership in one company never grants access to another.

## 9. Current-state caution
The initial setup previously showed RLS enabled on companies, branches, company_members, and profiles, with a limited set of policies. That is not proof that every target table or every role rule is secure. Re-inspect the live Supabase project and test the actual policies before production use.
