# dscr-ratio

**Debt service coverage ratio (DSCR)** — the single number that decides most
rental-property loans. DSCR = **net operating income ÷ annual debt service**.

`dscr-ratio` is a pure-Rust, zero-dependency crate that computes DSCR, the
amortizing monthly payment behind it, and a plain-language lender verdict. It
is the math behind the [DSCRRadar DSCR loan calculator](https://dscrradar.com/dscr-loan-calculator/),
a free tool for sizing real-estate investment loans.

## Why DSCR matters

For investment-property and rental loans, lenders underwrite on the **property's
cash flow**, not the borrower's personal income. DSCR is the headline ratio:

| DSCR     | Verdict     | What it means                                           |
|----------|-------------|---------------------------------------------------------|
| `< 1.00` | `fail`      | Rent does not cover debt service; most lenders decline.  |
| `1.00–1.15` | `marginal` | Barely covers debt; few programs, worse terms.        |
| `1.15–1.25` | `tight`    | Meets minimums; pricing improves above this band.     |
| `1.25–1.35` | `good`     | Standard "best terms" threshold for DSCR lenders.      |
| `≥ 1.35` | `strong`    | Comfortable cushion; best pricing and LTV.              |

A DSCR **≥ 1.25** is typically where DSCR lenders price their best terms;
below 1.00 the property does not cover its own loan from rent.

## Install

```toml
[dependencies]
dscr-ratio = "0.1"
```

## Quick start

```rust
use dscr_ratio::{monthly_payment, annual_debt_service, dscr, verdict};

// $100k loan, 7%, 30 years
let monthly = monthly_payment(100_000.0, 0.07, 360);   // ≈ 665.30
let debt    = annual_debt_service(monthly);            // ≈ 7,983.61
let ratio   = dscr(24_000.0, debt);                    // NOI / debt
println!("{}", verdict(ratio));                        // "good"
```

!!! tip "Run a full deal interactively"
    The crate handles the core math; for NOI line-items, reserves, and
    multiple scenarios use the
    [DSCRRadar DSCR loan calculator](https://dscrradar.com/dscr-loan-calculator/).

## Features

- **Pure Rust, no deps** — `f64` arithmetic, `#![no_std`]-friendly style.
- **DSCR** — `noi / annual_debt_service`, safe against non-positive debt.
- **Amortizing payment** — standard fixed-rate mortgage formula.
- **Lender verdict** — bucketed against common DSCR thresholds.
- **MIT licensed** — embed anywhere.

## Links

- Source crate: [`theluckystrike/dscr-ratio`](https://github.com/theluckystrike/dscrradar)
- Interactive tool: [DSCRRadar DSCR loan calculator](https://dscrradar.com/dscr-loan-calculator/)
