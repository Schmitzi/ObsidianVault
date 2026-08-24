# Resolution Sequence: Stages, Initiative Order, and Determinism

## The Stage pipeline, and how it maps (loosely) to the real Sequence of Play

```rust
pub enum Stage {
    SupplyDetermination,
    Movement,
    SupplyDistribution,
    Combat,
    Consumption,
    Attrition,
}
```

Be honest with yourself about what this is: a **simplified approximation**
of Rule 5.2's real Sequence of Play, not a faithful transcription of it. The
real sequence, per Operations Stage, is: Initiative Declaration → Weather →
Organization (reorg/construction/training, each with its own
completion/continuation steps) → Naval Convoy Arrival → Fleet → Reserve
Designation → Movement and Combat Phase (itself: Movement segment →
Breakdown Determination → Combat segment, which is *five* sub-steps —
Position Determination, Barrage, Retreat Before Assault, Force Assignment,
Anti-Armor, Close Assault — → Reserve Release) → Truck Convoy → Rail
Movement → Repair → Patrol.

The six-stage `Stage` enum was written *before* that section of the rules
had been read closely. It's being kept as the current skeleton because
Movement is built and tested against it — but expect this enum to grow
substantially once Combat gets implemented, since "Combat" as one stage is
nowhere near granular enough for the real Barrage → Retreat Before Assault →
Anti-Armor → Close Assault sequence. Flagging this now so the gap between
"what the code has" and "what the rules say" doesn't get mistaken for
already-correct.

## Why resolution is per-faction, in initiative order

```rust
pub fn resolve_turn(
    state: &GameState,
    order_sets: &[OrderSet],
    initiative_order: &[FactionId],
    seed: u64,
) -> (GameState, Vec<StageResult>) {
    ...
    for stage in stages {
        for &faction in initiative_order {
            let order_set = order_sets.iter().find(|os| os.faction == faction)...
            resolve_stage_for_faction(&stage, &mut current, order_set, &mut rng, &mut events);
        }
        ...
    }
}
```

This is the direct implementation of the [WEGO-vs-IGOUGO
resolution](./00-architecture.md#why-wego-got-revised-to-blind-submission-sequential-resolution)
decision: **for every stage**, faction A's order set is fully resolved
against the current state, *then* faction B's order set is resolved against
whatever state A's actions left behind. This is why one player moving first
can matter — if A moves a unit onto a hex, B's later movement in the same
stage sees that hex as occupied, exactly as the physical board game would
(A's counter is sitting there).

**What's simplified here, on purpose, for now:** the real rule lets Player A
run through the *entire* Movement-and-Combat loop possibly several times
(Continual Movement, 8.2) before B ever acts. The current code resolves one
`Stage` (e.g. all of Movement) for A, then all of Movement for B, then moves
to the next `Stage` — it doesn't yet support A doing Movement → Combat →
Movement → Combat repeatedly within a single Operations Stage before B gets
a turn. That's a real gap between the current code and 8.2, left for when
Combat exists (there's no point building repeatable Movement↔Combat
alternation before Combat resolves anything).

**What is NOT simplified:** order *submission*. Both factions author their
`OrderSet` blind, without seeing what the other submitted — `resolve_turn`
receives both order sets already-decided, and initiative order only affects
the sequence in which they're *applied*. This is what keeps the async,
24-hour-response-window multiplayer design intact even though resolution
itself is sequential rather than simultaneous.

## Determinism and RNG

```rust
pub struct TurnRng { rng: ChaCha8Rng }
impl TurnRng {
    pub fn from_seed(seed: u64) -> Self { ... ChaCha8Rng::seed_from_u64(seed) ... }
}
```

Two properties matter here, and losing either one breaks
[replay](./00-architecture.md#why-the-replay-system-is-orders-in-not-video):

1. **The seed must be server-generated**, never client-supplied. If a client
   could choose its own seed, it could pick one known (from prior testing)
   to produce favorable combat rolls. This is why `resolve_turn` takes
   `seed: u64` as a parameter from its caller (the server), rather than
   generating one internally — the server decides the seed once, stores it
   in the replay log alongside the orders, and passes it in.

2. **ChaCha8, not the default OS-entropy RNG.** Rust's `rand` crate's
   default source pulls from the operating system, which is *intentionally*
   non-reproducible (that's the whole point of OS entropy — for
   cryptographic randomness, you don't want two runs to match). `ChaCha8Rng`
   is a seeded, algorithmic PRNG: same seed in, same sequence of "random"
   numbers out, on every platform, forever. This is what makes replay
   possible at all — it's not that the game avoids randomness, it's that
   the randomness itself is made reproducible by construction.

Currently `TurnRng` is threaded through `resolve_stage_for_faction` as
`_rng` (unused, hence the compiler warning) — nothing draws from it yet
because Combat, the first stage that will actually need randomness (odds
tables, breakdown checks), isn't implemented. The plumbing exists ahead of
the feature that needs it so that when Combat is written, seeding is already
correct rather than being retrofitted (and potentially getting the
"server-generated, never client-supplied" property wrong under time
pressure).

## Why `StageResult` captures a full state snapshot per stage, not just events

```rust
pub struct StageResult {
    pub stage_name: &'static str,
    pub events: Vec<ResolvedEvent>,
    pub state_after: GameState,
}
```

This is more memory than strictly necessary for the replay log itself (which
only needs orders + seed, per the architecture doc). `state_after` exists
for **debuggability and UI**, not the core replay contract: if a turn
resolves in a way that looks wrong, being able to inspect the exact state
after *each* stage (not just before/after the whole turn) makes it possible
to pinpoint which stage introduced the problem, without re-deriving
intermediate states by replaying stages one at a time by hand. This is the
same idea floated early in the design as a "battle report" feature — showing
a player *why* a turn went the way it did, stage by stage — implemented here
as a side effect of how resolution is already structured, rather than a
separate system bolted on later.
