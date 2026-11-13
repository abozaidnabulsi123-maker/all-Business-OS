# AI Business OS — Technical Architecture

## 1. Product shape
AI Business OS is an Arabic-first, RTL, multi-tenant SaaS web application for commercial companies. The web experience is the first delivery target; a mobile application may follow later. English localization is planned after the Arabic foundation is stable.

## 2. Architectural principles
- Tenant isolation is mandatory: every company-owned record is scoped to a company.
- Authorization is enforced on the server and in the database, not only in the interface.
- Business transactions must be auditable and safely reversible.
- Financial and inventory posting must be atomic.
- AI features begin as read/analysis capabilities. High-impact actions require explicit human approval.
- Database migrations and documentation are version-controlled.
- The interface is responsive and designed for Arabic RTL from the start.

## 3. Logical layers
1. **Presentation:** responsive web UI, Arabic RTL layout, forms, tables, dashboards, and role-aware navigation.
2. **Application services:** validated business workflows for companies, sales, purchases, inventory, finance, reporting, and AI assistance.
3. **Identity and access:** authentication, company memberships, roles, and server-side authorization.
4. **Data:** Supabase Postgres with Row Level Security (RLS), migrations, constraints, indexes, and database functions.
5. **External services:** optional AI providers, notifications, file storage, and payment/subscription services. Integrate only when required and document credentials outside the repository.

## 4. Core domains
- Organization: companies, branches, memberships, profiles.
- Parties: customers, suppliers, contacts.
- Catalog: products, categories, units.
- Inventory: warehouses, stock balances, stock movements, transfers, adjustments.
- Sales: quotations, orders, invoices, returns, receipts.
- Purchasing: purchase requests, orders, supplier invoices, returns, payments.
- Finance: cash/bank accounts, journal entries, accounting periods, allocations.
- Operations: import/export, supply contracts, documents, alerts, audit events.
- Intelligence: dashboards, reports, forecasts, anomaly detection, AI recommendations.

## 5. Data and transaction rules
- Use UUID primary keys for tenant-owned entities unless an established schema requires otherwise.
- Add company_id to every company-owned table and branch_id where branch-level ownership is required.
- Use foreign keys and check constraints for data integrity.
- Store money in fixed-precision numeric columns with an explicit currency.
- Calculate authoritative totals on the server; never trust client-submitted totals.
- Post invoice, payment, journal, and stock effects in database transactions.
- Represent posted stock changes as append-only movements.
- Preserve document status history and audit sensitive changes.
- Use reversal, credit note, return, or adjustment workflows instead of silently deleting posted records.

## 6. Security boundaries
- Enable RLS on every tenant-owned table.
- Derive the active user's identity from the authenticated session.
- Verify company membership and required role for every read or write.
- Never accept a client-supplied user_id as proof of identity.
- Keep service-role secrets and provider keys out of browser code and Git.
- Record sensitive actions with actor, company, action, target, timestamp, and relevant metadata.

## 7. AI operating model
- Read-only analysis is the default.
- AI suggestions must cite the records or calculations they rely on.
- Actions such as posting invoices, changing stock, approving payments, deleting records, or changing permissions require explicit authorization and, where configured, human confirmation.
- Log AI-proposed and human-approved actions separately.
- Provide a way to review, reject, or reverse supported actions.

## 8. Environments and delivery
Maintain separate development and production environments. Apply schema changes through reviewed migrations. Before release, run authorization tests for at least: owner, admin, manager, accountant, employee, viewer, non-member, and unauthenticated user.

## 9. Important implementation note
This document defines the target architecture. It does not claim that every listed module or table is already implemented. Confirm the live Supabase schema and repository state before creating migrations or changing production data.
