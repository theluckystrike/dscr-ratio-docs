# Examples

Real-world DSCR scenarios using `dscr-ratio`. Every example is runnable Rust;
copy any block into `cargo run` or the [DSCRradar's examples dscrradar dscr loan](https://dscrradar.com/dscr-loan-calculator/)
to see the full line-item breakdown.

## 1. A clean single-family rental

$100k loan at 7% for 30 years, $2,000/mo rent, ~$600/mo operating expenses.

```rust
use dscr_ratio::{monthly_payment, annual_debt_service, dscr, verdict};

let principal = 100_000.0;
let rate      = 0.07;
let months    = 360;

let monthly_pmt = monthly_payment(principal, rate, months); // ≈ 665.30
let debt        = annual_debt_service(monthly_pmt);          // ≈ 7,983.61

// NOI = (rent - opex) * 12, debt service excluded
let gross_rent  = 2_000.0 * 12.0;   // 24,000
let opex        = 600.0  * 12.0;    //  7,200
let noi         = gross_rent - opex; // 16,800

let ratio = dscr(noi, debt);        // ≈ 2.10
println!("DSCR = {:.2} ({})", ratio, verdict(ratio)); // 2.10 (strong)
```

This deal clears most DSCR lender thresholds with room to spare.

## 2. Break-even property

NOI exactly equals annual debt service — DSCR = 1.00, verdict `marginal`.

```rust
use dscr_ratio::{dscr, verdict};

let ratio = dscr(20_000.0, 20_000.0);  // exactly 1.00
assert!((ratio - 1.0).abs() < 1e-9);
assert_eq!(verdict(ratio), "marginal"); // 1.00 is in the [1.00, 1.15) band
```

!!! warning "1.00 is not a pass"
    The crate buckets 1.00 as `marginal`, but in practice most DSCR lenders
    require **≥ 1.20–1.25**. A 1.00 deal leaves zero margin for vacancy,
    repairs, or rate increases.

## 3. Tight deal — sizing the max loan

Given a target DSCR of 1.25, what's the largest loan the NOI supports?
Work backwards: `annual_debt_service_max = noi / 1.25`, then solve for
principal.

```rust
use dscr_ratio::{dscr, monthly_payment, annual_debt_service, verdict};

let noi   = 18_000.0;
let rate  = 0.075;
let months = 360;
let target = 1.25;

// Max annual debt service the NOI can carry at target DSCR
let max_debt   = noi / target;                       // 14,400
let max_monthly = max_debt / 12.0;                   // 1,200

// Invert the amortization: principal that produces max_monthly at this rate/term
let r = rate / 12.0;
let n  = months as f64;
let max_principal = max_monthly * ((1.0 + r).powf(n) - 1.0)
                               / (r * (1.0 + r).powf(n));

// Verify
let check = dscr(noi, annual_debt_service(monthly_payment(max_principal, rate, months)));
println!("max loan ≈ ${:.0}, check DSCR = {:.2} ({})",
         max_principal, check, verdict(check));
// max loan ≈ $171,xxx, check DSCR = 1.25 (good)
```

## 4. Comparing two quotes

Same NOI, two different rate/term quotes — see which lands the better verdict.

```rust
use dscr_ratio::{monthly_payment, annual_debt_service, dscr, verdict};

let noi = 15_000.0;

let a = dscr(noi, annual_debt_service(monthly_payment(120_000.0, 0.075, 360)));
let b = dscr(noi, annual_debt_service(monthly_payment(120_000.0, 0.0825, 360)));

println!("7.50%: {:.2} ({})", a, verdict(a)); // ≈ 1.46 (strong)
println!("8.25%: {:.2} ({})", b, verdict(b)); // ≈ 1.39 (strong)
```

Both qualify; the lower rate widens the cushion.

## 5. Zero-interest / interest-only edge case

`monthly_payment` handles a zero rate as straight-line principal, so a
$120k, 0%, 360-month "loan" just returns $333.33/mo — useful for modeling
seller financing or deferred balances.

```rust
use dscr_ratio::monthly_payment;

assert!((monthly_payment(120_000.0, 0.0, 360) - 333.3333).abs() < 1e-3);
```

---

### Next steps

For full underwriting — reserves, vacancy assumptions, multiple rate sheets —
paste your NOI and loan terms into the
[the DSCRradar page on examples dscrradar dscr loan](https://dscrradar.com/dscr-loan-calculator/).
