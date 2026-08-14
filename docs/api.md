# API Reference

`dscr-ratio` exposes four free functions. All take and return `f64` (and one
`u32` term). No structs, no traits to implement, no features to enable.

## `dscr`

```rust
pub fn dscr(noi: f64, annual_debt_service: f64) -> f64
```

Debt service coverage ratio = **NOI ÷ annual debt service**.

- Returns `0.0` when `annual_debt_service <= 0.0` (avoids division by zero).
- `noi` is annual net operating income (rent minus operating expenses,
  before debt service and before taxes).

```rust
use dscr_ratio::dscr;

assert!((dscr(24_000.0, 20_000.0) - 1.2).abs() < 1e-9);
assert!((dscr(20_000.0, 20_000.0) - 1.0).abs() < 1e-9); // break-even
assert_eq!(dscr(100_000.0, 0.0), 0.0);                   // no debt → 0
```

## `monthly_payment`

```rust
pub fn monthly_payment(principal: f64, annual_rate: f64, months: u32) -> f64
```

Monthly payment on a **fully-amortizing fixed-rate loan** (the standard US
mortgage amortization formula).

- `annual_rate` is the nominal annual rate (e.g. `0.07` for 7%); it is
  divided by 12 internally.
- `months` is the loan term in months (`360` = 30 years).
- Returns `0.0` when `months == 0`.
- When `annual_rate == 0.0`, returns the straight-line
  `principal / months`.

$$
M = P \cdot \frac{r(1+r)^n}{(1+r)^n - 1}
\quad\text{where}\quad
r = \frac{\text{annual rate}}{12},\;\; n = \text{months}
$$

```rust
use dscr_ratio::monthly_payment;

// $100k, 7%, 30y → ≈ $665.30/mo
let m = monthly_payment(100_000.0, 0.07, 360);
assert!((m - 665.30).abs() < 0.1);
```

## `annual_debt_service`

```rust
pub fn annual_debt_service(monthly: f64) -> f64
```

Annual debt service from a monthly payment — simply `monthly * 12.0`.
This is the denominator of `dscr`.

```rust
use dscr_ratio::{monthly_payment, annual_debt_service};

let debt = annual_debt_service(monthly_payment(100_000.0, 0.07, 360));
// ≈ 7,983.61
```

## `verdict`

```rust
pub fn verdict(ratio: f64) -> &'static str
```

Plain-language verdict for a DSCR value, matching common DSCR lender
thresholds:

| DSCR range        | Return value | Meaning                                  |
|-------------------|--------------|------------------------------------------|
| `ratio < 1.00`    | `"fail"`     | Rent does not cover debt.                |
| `1.00 ≤ r < 1.15` | `"marginal"` | Barely covers debt.                      |
| `1.15 ≤ r < 1.25` | `"tight"`    | Meets most minimums.                     |
| `1.25 ≤ r < 1.35` | `"good"`     | Best-terms threshold for most lenders.   |
| `ratio ≥ 1.35`    | `"strong"`   | Comfortable cushion, best pricing.       |

```rust
use dscr_ratio::verdict;

assert_eq!(verdict(0.9), "fail");
assert_eq!(verdict(1.1), "marginal");
assert_eq!(verdict(1.2), "tight");
assert_eq!(verdict(1.3), "good");
assert_eq!(verdict(1.5), "strong");
```

!!! note "Verdict vs. lender policy"
    These buckets reflect the **most common** DSCR lender thresholds, but every
    lender sets its own cutoffs and overlays. For a quote on a specific deal,
    run the numbers through the
    [DSCRradar's api dscrradar dscr loan](https://dscrradar.com/dscr-loan-calculator/).
