# CPA and Movement: Why It's a Soft Cap, and How `resolve_move_order` Works

## What CPA actually is, per the rules

Section 6.0 introduces the Capability Point Allowance: a number each unit has that gets spent on *everything* — movement, combat, construction, loading — not a separate "movement points" pool like most wargames use. Section 8.2 ("Continual Movement") then makes a specific, unusual claim:

<i>...a unit does not possess a set Movement Allowance which it may never exceed, as in most wargames. Rather a unit has a Capability Point Allowance. If a Phasing Player wishes to push his units past their CPA he may do so, knowing that it will affect that unit's Cohesion. This is a **soft cap**, not a hard one. Most wargames: you have N movement points, you stop at 0, end of story. CNA: you *can* keep going past your CPA — you're trading Cohesion (eventually Morale) for the extra reach. A unit desperate to reach a battle, or retreating from one, can push itself further than it "should," at a cost paid later.</i>

## Why the code models this as tracked overspend, not a blocked action

```rust
pub cpa_base: Fixed,
pub cp_expended_this_stage: Fixed,
```

`resolve_move_order` never refuses to move a unit for being low on CP. It just keeps accumulating `cp_expended_this_stage`, and the moment that number crosses `cpa_base`, it emits `ResolvedEvent::CpaExceeded { unit, overage }`. That event exists specifically so the *Attrition* stage can look at it and apply whatever the real Cohesion penalty formula turns out to be (6.2, not yet read). This is the pattern worth noticing: rather than hard-coding "and here's the penalty" into `Movement`, the code separates "did the rule-violation-that-isn't-really-a-violation happen" (recorded as an event, in Movement) from "what does it cost" (applied later, in Attrition). That separation matches the rulebook's own structure — CPA overspend is a movement-time fact, but Cohesion/Morale consequences are their own system (17.0) layered on top.

## Walking through `resolve_move_order` step by step

For each hex in the ordered path:

1. **Contiguity check (8.13).** `position.is_adjacent_to(next_hex)` — units can't skip hexes. If this fails, movement halts right there; nothing after this point in the path is processed. This matters for realism: a client should never be able to submit a path with a gap and have the server silently "fix" it — an invalid path is a bug (or cheat attempt) worth surfacing, not quietly repairing.

2. **Enemy-occupied hex check (8.13).** `hex_occupied_by_enemy` scans all units for one of a different faction sitting on `next_hex`. If found, movement halts *before* entering it — the rule says you can never enter a hex containing an enemy unit, full stop (with an exception for Desert Raiders, 27.4).

3. **Fuel, once per Operations Stage (32.23).** This is the part that surprised the initial design the most — see the next section.

4. **Terrain cost lookup and CPA accounting.** `hex_entry_cost` (below) returns `None` for impassable terrain (halts movement) or `Some(cost)`. The cost is added to `cp_expended_this_stage`, and if that push crosses `cpa_base` for the first time this move, a `CpaExceeded` event fires.

Notice the function takes ownership of local `position`/`cp_expended`/`moved_this_stage` variables and only writes them back to the actual `Unit` in `state.units` *once*, at the very end. This is a deliberate Rust borrow-checker-friendly pattern: mutating `state.units.get_mut(&unit_id)` inside the loop while also needing to read `state.map` and scan `state.units` for enemy occupancy (immutable borrows) in the same loop would conflict. Working with local copies and writing back once sidesteps that — and has the side benefit of making the function easier to reason about, since the "in-progress" state during path-walking is just plain local variables, not live mutations to shared state that some other code could observe mid-loop.

## Why fuel is "once per stage," not "per hex" — and why that was a surprise

The very first version of this engine modeled fuel as continuous drain: `fuel_per_turn_moving`, subtracted every turn a unit moved. That was a reasonable-sounding default *before* reading 32.23:

<i> Fuel Points are expended per Operations Stage. Basically, each battalion-equivalent expends one Fuel Point in any Operations Stage in which it is moved by land... the required one (or two) Fuel Points are expended from a Supply Unit. This unit may now be moved up to its CPA in any combination of movements and halts within that Operations Stage.</i>

Fuel here is closer to "unlocking the ability to move this stage" than "paying per kilometer." A unit that moves 1 hex and a unit that moves 8 hexes (using its full CPA) pay the *same* Fuel Point cost, as long as neither exceeds their CPA — only exceeding CPA triggers *additional* fuel expenditure. This is why `moved_this_operations_stage: bool` exists on `Unit` — it's not a minor implementation detail, it's the direct representation of "has this unit already paid its one-time stage fee."

`fuel_points_for()` returns 2 Fuel Points specifically for `MovementClass::ArmorBattalion`, 1 for everything else — straight from 32.24's differentiated costs by unit type.

## Where fuel comes from: the honest gap (32.16)

```rust
fn try_spend_fuel(state: &mut GameState, faction: FactionId, needed: Fixed) -> bool {
    for node in state.supply_nodes.values_mut() {
        if node.controlling_faction == Some(faction) && node.stockpile_fuel >= needed {
```

This grabs fuel from **any** friendly Supply Node anywhere on the map with enough stockpile. That is *not* what the rules say. 32.16 requires the Supply Unit be **within range** — specifically, within half the drawing unit's CPA, traced as though by a medium truck (or as infantry movement for non-motorized units), and that the supply line not cross impassable terrain or an unoccupied enemy Zone of Control.

This was left as a deliberate, marked simplification rather than guessed at, because implementing it properly needs a pathfinding/reachability check (essentially: can I trace a valid line of hexes from this unit to a supply node, within a CP budget, avoiding certain terrain/ZOCs) that's its own piece of work, not a one-line fix. The `TODO (32.16)` comment marks exactly where that logic needs to slot in later — probably as part of the `SupplyDetermination` stage, which currently does nothing, computing *which* nodes are actually in range for each unit *before* Movement runs, rather than Movement doing an unbounded search itself.

## The Terrain Effects Chart gap

`hex_entry_cost()` in `resolve.rs` currently has exactly two real numbers, both quoted directly in rule *text* rather than read off the chart itself:

- 1/4 CP for roads (motorized units) — 8.31
- +8 CP moving *down* an escarpment via track (vehicles) — 8.31

Everything else (open desert base cost, salt marsh, oasis, the "up" escarpment cost, non-road/track terrain in general) is a placeholder, clearly commented as such. The real chart (8.37) is a printed numeric table that this PDF's OCR rendered as garbage — same failure mode as the Order-of-Battle charts at the document's end. Until that page is manually transcribed (or a clean copy found), any balance/testing conclusions drawn from movement costs should be treated as provisional.
