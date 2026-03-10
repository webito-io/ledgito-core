# Ledgito Core

> Open-source financial back-office engine for centralized cryptocurrency exchanges.

Ledgito is a self-hosted, double-entry accounting system purpose-built for CEX operators. It gives you real-time visibility into your exchange's financial position — what you own, what you owe to users, and whether you're solvent — without paying $450/month for a SaaS tool.

Built and maintained by [Webito](https://github.com/webito-io).

---

## The Problem

Most crypto exchanges manage their finances with spreadsheets and manual snapshots. Every night someone records the USDT price, aggregates user balances, and tries to figure out if the exchange made money today. This is slow, error-prone, and impossible to audit.

Enterprise tools like Cryptio or Coincile exist, but they're cloud-only, expensive, and built for Western institutional clients — not for the growing number of mid-sized exchanges in emerging markets.

**Ledgito fills that gap.**

---

## Features

- **Double-Entry Ledger** — Every fund movement creates a balanced Debit/Credit pair. No money appears or disappears without a trace.
- **Real-Time Position Monitor** — Continuously compares total exchange assets against total user liabilities. Alerts you before you're in trouble.
- **Multi-Venue Aggregation** — Connect to Binance, OKX, on-chain wallets, and your own matching engine simultaneously.
- **Reconciliation Engine** — Detects mismatches between your internal (virtual) balances and real on-chain/exchange balances.
- **P&L Reporting** — Daily, weekly, and monthly profit & loss broken down by fee income, gas expenses, and operational costs.
- **Audit Trail** — Immutable, append-only ledger entries. Nothing is ever deleted or edited — only reversed.
- **Self-Hosted** — Your financial data stays on your infrastructure. No vendor lock-in.
- **Docker-ready** — Single `docker-compose up` to get started.

---

## Architecture

```
ledgito-core/
├── cmd/
│   └── server/
│       └── main.go                 # Entry point
├── internal/
│   ├── ledger/                     # Core double-entry engine
│   │   ├── domain/                 # Entities: Account, JournalEntry, Transaction
│   │   ├── repository/             # DB queries (sqlc generated)
│   │   ├── service/                # Business logic
│   │   └── handler/                # HTTP handlers (Fiber)
│   ├── position/                   # Real-time exposure & short detector
│   │   ├── service/
│   │   └── handler/
│   ├── valuation/                  # Price oracle, base currency conversion
│   │   └── service/
│   ├── integrations/               # Exchange & blockchain adapters
│   │   ├── binance/
│   │   ├── okx/
│   │   └── blockchain/             # ETH, TRX, BTC indexers
│   ├── reconciliation/             # Virtual vs real balance comparison
│   │   └── service/
│   └── reporting/                  # Balance sheet, P&L, audit exports
│       ├── service/
│       └── handler/
├── pkg/
│   ├── decimal/                    # Precision math wrapper (shopspring/decimal)
│   ├── logger/                     # Structured logging (zap)
│   └── middleware/                 # Auth, rate limiting, request ID
├── db/
│   ├── migrations/                 # golang-migrate SQL files
│   ├── queries/                    # Raw SQL queries (sqlc input)
│   └── sqlc/                       # Auto-generated type-safe Go code
├── config/
│   └── config.go                   # Env-based config (viper)
├── docker-compose.yml
├── Makefile
└── README.md
```

**Pattern:** Clean Architecture — each module has its own domain, repository, service, and handler layers. No magic, no hidden DI — explicit and auditable.

**Stack:**

| Layer | Technology | Why |
|---|---|---|
| Framework | [Fiber](https://gofiber.io) | Express-like API, fastest Go web framework |
| Database | PostgreSQL 15+ | ACID compliance, NUMERIC precision |
| Query Layer | [sqlc](https://sqlc.dev) | Write SQL, get type-safe Go — no ORM magic |
| Migrations | [golang-migrate](https://github.com/golang-migrate/migrate) | Simple, reliable |
| Precision Math | [shopspring/decimal](https://github.com/shopspring/decimal) | Prevents floating-point errors in financial calc |
| Config | [viper](https://github.com/spf13/viper) | Env + file config |
| Logging | [zap](https://github.com/uber-go/zap) | Structured, high-performance logging |
| Container | Docker + docker-compose | One-command setup |

---

## Why Go?

Go is increasingly the language of choice for financial infrastructure — Coinbase, Binance, and most major exchanges run their core systems on Go. It compiles to a single binary, has a tiny Docker footprint, and its explicit error handling is a feature, not a bug, when you're building systems where silent failures cost real money.

---

## How It Works

### The Core Equation

```
Net Position = Total Exchange Assets − Total User Liabilities

Coverage Ratio = (Assets / Liabilities) × 100
```

| Coverage | Status | Action |
|---|---|---|
| ≥ 110% | Healthy | — |
| 100–110% | Watch | Monitor closely |
| 90–100% | Warning | Act immediately |
| < 90% | Critical | Consider halting withdrawals |

### Example: User Deposits 1 ETH

| Account | Debit | Credit |
|---|---|---|
| Hot Wallet ETH *(Asset)* | 1.0 ETH | — |
| User Balance ETH *(Liability)* | — | 1.0 ETH |

Assets go up, liabilities go up by the same amount. The exchange is neither richer nor poorer — but now it's tracked.

### Example: User Pays a Trading Fee

| Account | Debit | Credit |
|---|---|---|
| User USDT Balance *(Liability)* | 2 USDT | — |
| Fee Income *(Income)* | — | 2 USDT |

Now the exchange is actually richer. This is how P&L accumulates.

---

## Roadmap

| Phase | Scope | Status |
|---|---|---|
| **Phase 1** | Core double-entry ledger, Chart of Accounts, REST API | 🔲 In progress |
| **Phase 2** | Position Monitor, Coverage Ratio, alert system | 🔲 Planned |
| **Phase 3** | Exchange adapters (Binance, OKX), blockchain indexers | 🔲 Planned |
| **Phase 4** | Reporting dashboard, P&L, Excel/PDF export | 🔲 Planned |
| **Phase 5** | Audit trail hardening, Proof of Reserves, RBAC | 🔲 Planned |

---

## Getting Started

```bash
git clone https://github.com/webito-io/ledgito-core.git
cd ledgito-core
cp .env.example .env
docker-compose up -d
```

API will be available at `http://localhost:3000`.

> **Note:** Full setup documentation is coming with the Phase 1 release.

---

## Contributing

Ledgito is in early development. If you run a crypto exchange and want to shape the roadmap, open an issue and tell us about your setup.

We welcome contributions of all kinds — bug reports, feature requests, documentation, and code.

Please read [CONTRIBUTING.md](./CONTRIBUTING.md) before submitting a pull request.

---

## License

[MIT](./LICENSE) — free to use, modify, and self-host.

---

<p align="center">
  Built with care by <a href="https://github.com/webito-io">Webito</a>
</p>
