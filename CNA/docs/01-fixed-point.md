# Fixed-Point Math: Why `Fixed` Exists and How It Works

## The problem

`f32`/`f64` (floating point) do not guarantee identical results across different machines, compilers, or optimization levels for the same arithmetic. This is a real, documented property of IEEE 754 floating point. Not a Rust-specific quirk. The rounding of a floating-point operation can differ based on things like whether the CPU uses an intermediate 80-bit register, or whether the compiler fuses a multiply-add into one instruction.

Why this matters *specifically* for CNA: [replay](./00-architecture.md#why-the-replay-system-is-orders-in-not-video) only works if `resolve_turn()` produces byte-identical results every time, forever, on every platform. If fuel consumption were computed with `f64`, two players' machines could compute a unit's fuel as `59.999999998` and `60.000000001` respectively — both "correct" by float standards, but *different*, and that difference compounds turn over turn until states diverge and replays desync. This class of bug is notoriously hard to track down after the fact, because it doesn't reproduce reliably — which is exactly why it's worth designing it out from the start instead of hoping it doesn't happen.

## The fix: represent everything as scaled integers

```rust
const SCALE: i64 = 10_000;

pub struct Fixed(i64);
```

A `Fixed` doesn't store "59.75 fuel points" as the float `59.75`. It stores the *integer* `597_500` (i.e., `59.75 * SCALE`). Integer arithmetic on a fixed CPU architecture is exact and reproducible — `597_500 + 25_000` is `622_500`, full stop, on every machine that has ever existed. There's no rounding ambiguity because there's no fractional representation to round. `SCALE = 10_000` gives four decimal digits of precision, which comfortably covers things like "1/4 CP" (`Fixed::from_fraction(1, 4)` → internally `2_500`) without any rounding loss for the granularity this game needs.

## Why methods instead of operator overloading (`+`, `-`)

```rust
pub fn checked_add(self, rhs: Fixed) -> Option<Fixed>
pub fn saturating_sub(self, rhs: Fixed) -> Fixed
```

If `Fixed` implemented `std::ops::Add`, writing `a + b` would silently do *something* on overflow (`i64` overflow is a panic in debug builds, silent wraparound in release builds — genuinely different behavior between the two, which is its own trap). By requiring `checked_add` or `saturating_sub` explicitly, every call site has to decide on purpose what should happen:

- `checked_sub` returning `None` — appropriate somewhere you want to *reject* an operation that would go negative (e.g. "can I afford to spend this many CPs" — if not, don't spend them).
- `saturating_sub` — appropriate where negative doesn't make physical sense and clamping to zero is fine (e.g. `cpa_base.saturating_sub(cp_expended)` in `Unit::cp_remaining()` — if you've overspent, "remaining" is just 0, not a negative number the caller then has to remember to clamp themselves).

This is a small amount of extra typing at every call site, in exchange for every overflow/underflow decision being visible in the code rather than buried in a default.

## Why there's no `From<f64>` on the "hot path"

`to_f64()` exists — but only for *display*. There is deliberately no `Fixed::from(some_f64)` used anywhere inside `engine`'s simulation logic. The one-way door matters: once a float has entered the calculation, you've already lost the guarantee. Keeping the conversion function named `to_f64` (rather than implementing the `Into`/`From` traits, which Rust would let you call implicitly in more places) makes every conversion point `grep`-able — you can always find every place a `Fixed` became a float by searching for `to_f64`, which matters when auditing "did a float sneak into resolution logic somewhere."

## Where this will pay off later

Any time I'm tempted to write `0.25` or `1.5` directly in engine code, I can use `Fixed::from_fraction(1, 4)` or `Fixed::from_int(1)`. If a calculation needs a ratio that doesn't reduce cleanly, that's a sign to widen `SCALE` or introduce a new fixed-point helper method — not to reach for a float "just this once."
