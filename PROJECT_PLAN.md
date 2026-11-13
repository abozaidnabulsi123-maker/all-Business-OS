# AI Business OS — Project Plan

## Vision
An Arabic-first, multi-company business operating system for commercial companies. It unifies operations, financial records, inventory, reporting, and AI-assisted workflows in one secure SaaS platform.

## Product decisions
- Product: AI Business OS.
- Market: commercial companies across industries.
- First client: responsive web application; mobile app later.
- First language: Arabic with RTL; English planned.
- Architecture: multi-tenant SaaS with strict company data isolation.
- Delivery: incremental, tested modules with security built in.

## Core modules
1. Companies, branches, memberships, roles, and permissions.
2. Customers, suppliers, contacts, and account balances.
3. Products, categories, units, warehouses, and stock movements.
4. Sales quotations, orders, invoices, returns, and receipts.
5. Purchase requests, orders, invoices, returns, and payments.
6. Cash/bank accounts, journals, and financial reports.
7. Import, export, supply, contracts, and document tracking.
8. Dashboards, operational reports, alerts, and audit history.
9. AI assistant for analysis, summaries, forecasts, anomaly detection, and workflow suggestions.
10. Subscription, tenant settings, localization, and administration.

## Delivery phases
- Phase 0: verify repository, environments, migrations, authentication, tenant model, RLS, and automated checks.
- Phase 1: company setup, main branch, invitations, memberships, role assignment, and access tests.
- Phase 2: customers, suppliers, products, units, warehouses, and opening balances.
- Phase 3: sales/purchase document lifecycle, line items, taxes, discounts, receipts, payments, and returns.
- Phase 4: stock ledger, valuation policy, cash/bank ledger, journals, reconciliation, and period controls.
- Phase 5: dashboards, reports, AI analysis, recommendations, and human-approved actions.
- Phase 6: security review, backups, monitoring, performance, onboarding, and subscription controls.

## Critical business rules
- Every business record belongs to a company; branch scope applies where relevant.
- Posted financial and stock documents are corrected through reversal/return workflows, not silent destructive edits.
- Invoice totals are calculated and validated on the server.
- Stock changes are represented by immutable movements.
- Financial and inventory posting must be atomic.
- AI cannot independently approve payments, delete records, or post high-impact transactions.
- Sensitive actions are authorized server-side and recorded in an audit log.

## Definition of done
A feature is complete only when its schema/migration, authorization policies, server validation, UI states, error handling, tests, and documentation are in place. A proposed schema is not considered deployed until verified in the target Supabase project.
