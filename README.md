# ERP Platform

A modular ERP that runs HR, accounting, inventory, sales, purchasing and POS for multiple companies from one platform. Every financial movement is posted to a double-entry ledger, and every action leaves an audit trail.

> **Status:** core modules are built (76 of 78 checklist items done). Still open: sales reports and PDF/Excel export for reports. See [TODO.md](TODO.md).

<!-- Add screenshots here: dashboard, sales invoice, journal entry, POS screen. -->

## Modules

| Module | What it covers |
|---|---|
| **HR & Payroll** | Departments, positions, employees, leave types and requests, attendance, payroll runs |
| **Accounting** | Chart of accounts, journal entries with balanced debit/credit lines, multi-currency |
| **Inventory** | Warehouses, products, stock movements on an immutable stock ledger |
| **Sales** | Customers and sales invoices; confirming an invoice posts its journal entry automatically |
| **Purchasing** | Vendors and goods received notes (GRN); confirming a GRN updates stock and the ledger |
| **POS** | Point-of-sale transactions linked to inventory and accounting |
| **Reports & Dashboard** | Financial reports and a KPI dashboard |

## Roles

Super Admin · Accountant · HR Manager · Warehouse · Sales Rep · Purchasing · Viewer (read-only reports). Each role only sees the modules it needs, enforced with spatie/laravel-permission.

## Design decisions

Full write-up in [Architecture.md](Architecture.md).

1. **Money is stored as integers (cents)**, never floats, so totals never drift from rounding.
2. **Immutable stock ledger:** stock levels come from recorded movements, not from editing a quantity field.
3. **Double-entry is enforced in code:** an unbalanced journal raises `UnbalancedJournalException`, and overselling raises `InsufficientStockException`.
4. **Modules talk through services**, so Sales confirming an invoice calls Accounting's service instead of writing to its tables directly.
5. **Side effects run on queues** (Redis), so heavy work doesn't block the request.
6. **Soft deletes everywhere**, so financial history is never lost.

## Architecture

```
backend/app/
├── Base/            BaseController, BaseRepository, BaseService
├── Modules/
│   ├── Accounting/  HR/  Inventory/  POS/
│   └── Purchasing/  Reports/  Sales/  Auth/
└── Helpers/         MoneyHelper

Request → Controller → Service (business rules) → Repository (data access) → Model
```

## Tech stack

| Layer | Stack |
|---|---|
| Backend | Laravel 12 (PHP 8.2+), Sanctum, spatie/laravel-permission, DomPDF, Laravel Excel |
| Frontend | Next.js 14 (App Router), React 18, TypeScript, Tailwind CSS, shadcn/ui, TanStack Query, react-hook-form + zod, Recharts |
| Data | MySQL, Redis (queues) |

## Run it locally

```bash
# API
cd backend
composer install
cp .env.example .env          # MySQL and Redis connection
php artisan key:generate
php artisan migrate --seed    # roles, admin user, sample company and data
php artisan serve
php artisan queue:work        # separate terminal

# Frontend (separate terminal)
cd frontend
npm install
cp .env.example .env.local    # NEXT_PUBLIC_API_URL
npm run dev
```

## Project docs

[PRD.md](PRD.md) (requirements) · [Architecture.md](Architecture.md) (flows and decisions) · [database.md](database.md) (schema) · [TODO.md](TODO.md) (build checklist) · [GUIDE.md](GUIDE.md) (phase-by-phase build guide)
