# The Terrain Effects Chart (8.37): Transcription and Modeling Decisions

## **One placeholder turned out to be actively wrong, not just approximate:**
The original design guessed "Up Escarpment costs +10 CP for vehicles," extrapolating from the real "+8 down" number in the rule text. The actual chart says Up Escarpment is **Prohibited** to vehicles — not expensive, *impossible* without a track. That's a difference in kind, not degree: code built on the guess would have let tanks scale escarpments at a high CP cost, when the real rule is that they categorically cannot. This is the concrete argument for treating anything derived from an OCR gap as provisional until verified — a wrong number that's merely "off" is easy to spot when playtesting feels miscalibrated; a wrong number that changes what's *possible* produces behavior that looks intentional right up until someone who knows the real rules points it out.

## The full chart, transcribed

**CP Cost to Enter or Cross** (non-Motorized / Motorized), **Breakdown
Value**, and **Stacking Limit**:

| Terrain Type              | non-Motortized         | Motorized          | Breakdown | Stacking |
| ------------------------- | ---------------------- | ------------------ | --------- | -------- |
| Clear                     | 2                      | 2                  | 4         | 6        |
| Gravel                    | 2                      | 2                  | 6         | 6        |
| Salt Marsh²               | 3                      | 2                  | 6         | 6        |
| Heavy Vegetation          | 3                      | 3                  | 3         | 6        |
| Rough                     | 3                      | 4                  | 8         | 6        |
| Mountain                  | 4                      | 6                  | 12        | 3        |
| Delta                     | 2                      | 4                  | 2         | 6        |
| Desert                    | 3                      | 4³                 | 24        | 6        |
| Major City                | 1                      | ½                  | ½         | 8        |
| Swamp⁴                    | road/railroad only     | road/railroad only | —         | 6        |
| Village/Bir/Oasis         | same as terrain in hex |                    |           |          |
| Railroad⁵                 | same as terrain in hex |                    |           |          |
| Road⁶                     | 1                      | ½                  | ½         | 5⁷       |
| Track⁸                    | 1                      | 1                  | —         | 5⁷       |
| Ridge                     | +2                     | +4                 | 2         | —        |
| Up Slope                  | +2                     | +4                 | 2         | —        |
| Down Slope                | +1                     | +2                 | 2         | —        |
| Up Escarpment             | +6                     | **P**              | —         | —        |
| Down Escarpment           | +4                     | +8⁹                | +6        | —        |
| Wadi                      | +1¹⁰                   | +4¹⁰               | +8        | —        |
| Major River               | +8                     | **P**¹¹            | —         | —        |
| Minor River               | +3                     | +6                 | +1        | —        |
| Fortifications (L1/L2/L3) | same as terrain in hex |                    |           |          |
| Friendly Minefield¹³      | +1                     | +4                 | 0         | —        |
| Enemy Minefield¹³         | +4                     | +CPA               | +2        | —        |

`P` = Prohibited. `CP` = Capability Point. `Mot` = Motorized. Superscripts
are footnote references, below.

**Combat Adjustment columns** (Barrage / Anti-Armor / Close Assault) exist on the chart too — these are column shifts on the relevant Combat Results Table, not movement costs, so they're not implemented yet (Combat doesn't exist in the engine). Transcribing them here so they don't need re-photographing later:

| Terrain | Barrage | Anti-Armor | Close Assault |
|---|---|---|---|
| Rough | L1 | L1 | L2 |
| Mountain | L2 | L2 | L3 |
| Heavy Vegetation | – | L1 | L1 |
| Ridge | – | L2 | L2 |
| Up Slope | – | L1 | L2 |
| Down Slope | – | L1 | R1 |
| Up Escarpment | – | P | L3 |
| Down Escarpment | – | L2 | R1 |
| Wadi | – | – | L1 |
| Major River | – | – | L6 |
| Minor River | – | – | L2 |
| Fortifications L1/L2/L3 | L1¹²/L2¹²/L2¹² | L1/L2/L2 | L2/L3/L4 |
| Friendly Minefield | – | L1 | L1 |
| Major City | see Fortifications | | |

## Footnotes

1. A garrison unit in its assigned hex occupies zero stacking space. An unlimited number of anti-aircraft units may occupy a Major City hex at zero tacking space; three AA units may occupy an Airfield hex at zero stacking space; one AA unit may occupy an Air Landing Strip hex at zero stacking space.
2. Vehicles *other* than Light Trucks, Recce-type units, and motorized infantry may only enter/leave a Salt Marsh hex on a track. Motorized units may not engage in combat into/out of a Salt Marsh (Case 8.44).
3. Light Trucks, motorcycle reconnaissance units, and motorcycle infantry units may not enter Desert, even via a track.
4. Alexandria and Cairo hexes are Level Three Fortifications; all others are Level Two.
5. Commonwealth units may be able to use Rail Movement (Case 8.7).
6. Negates all hexside terrain feature entry costs and Breakdown Point Values.
7. Applies only to the units/truck points in that hex at that time.
8. Halves all terrain feature entry costs and all hex/hexside Breakdown Point values *except* the Capability Points expended and the Breakdown acquired by moving a vehicle down an escarpment.
9. Motorized units and trucks may only cross [an escarpment] on a track.
10. May only be crossed during a Rainstorm if the hexside is crossed by a road or railroad (treated as a road for this purpose), at a cost of two additional Capability Points.
11. May only be crossed if the hexside is crossed by a road or railroad. A motorized unit may so cross at no additional cost in Capability or Breakdown Points.
12. Armor targets receive the defensive benefit only if the hex is a Major City hex.
13. Engineer units reduce cost to enter (Case 26.2). If assaulting forces are in an Enemy minefield, the non-Phasing forces receive L1 shifts for Anti-Armor and Close Assault if not already receiving them for occupying a Friendly minefield.

## Modeling decisions and where they're a judgment call, not a fact

### Village/Bir/Oasis/Port became an `Overlay`, not a `Terrain`

The chart is explicit — these are "same as terrain in hex for all purposes." A hex with an Oasis marker still costs whatever its *actual* terrain (Clear, Desert, whatever) costs to enter. Modeling them as their own `Terrain` variant would have meant inventing a cost for them that the chart never gives, and would have made "what does this hex actually cost" ambiguous between two representations of the same fact. `HexInfo` now carries `terrain: Terrain` (cost-bearing) and `overlay: Option<Overlay>` (informational — matters for supply/capture/garrison logic later, not movement cost) as two separate fields specifically so there's only one source of truth for cost.

### Road negates other hexside features; Track does too — with one named exception, which forced a data model change

Case [8.33] settles what footnote 8's ambiguous "halves" wording left
unclear:

<i>Units which are moving along Roads or Tracks ignore, for movement purposes, any other terrain in the hex or hexside, **with the exception of vehicles crossing Escarpments**.</i>

This confirms Track behaves exactly like Road for movement cost — a flat override, not a halving calculation — with exactly one named exception: Escarpments. And that exception, combined with footnote 9 ("Motorized units and trucks may only cross [an escarpment] on a track"), reveals something important: **a hexside can have a Track and an Escarpment on it at the same time**, and that's not a rare edge case — it's the *only* way a vehicle can ever descend an escarpment at all. A vehicle needs the Track to be allowed to cross, and still pays the Escarpment's own cost rather than the Track's usual flat rate.

The original `HexsideFeature` design was one mutually-exclusive enum per hexside — `Road` *or* `Track` *or* `Escarpment`, never two at once. That data model was structurally incapable of representing the one case the rules actually describe. This is why `HexsideFeature` became a struct of three independent `Option` fields (`route`, `elevation`, `water`) instead of one enum — the fix wasn't in the cost *function*, it was one layer down, in what the *data* was even able to say. Worth internalizing as a general lesson: sometimes a rule turns out to be describing a state combination your types don't allow, and the fix is upstream of any logic bug.

`hex_entry_cost` now checks Escarpment presence *before* the generic Road/Track override, specifically because it's the named exception to that override — the ordering in the code is the ordering the rule implies, not an arbitrary implementation choice.

### Swamp and Major River narrow the general rule further — Track doesn't count for either

Two more chart-specific rules turned out to be *narrower* than 8.33's general statement, not broader: Swamp's row says "enter only on road or **railroad**," and Major River's footnote 11 says "crossed if crossed by a road or **railroad**." Neither mentions Track. Since 8.33 is the general rule and these are specific chart entries, the specific wording wins — Track alone does not make Swamp or Major River passable to vehicles, even though it unlocks nearly everything else. `hex_entry_cost` checks both of these before the generic override for exactly this reason.

**Railroad-only access remains an open gap for both.** Neither Swamp nor Major River has a chart-given numeric cost for a bare Railroad hexside (no Road), and Railroad's own row gives it no discount for ordinary CP movement either (that only exists under 8.7 Rail Movement). Both currently return `None` (impassable) for Railroad-only access — the conservative reading, still flagged rather than guessed at.

### Enemy Minefield's "+CPA" cost isn't implemented yet

The chart lists a Motorized unit's cost to enter an Enemy Minefield as `+CPA` — not a number, but "your entire remaining CPA." That's a fundamentally different kind of cost than everything else on this chart: every other entry is a fixed value independent of the moving unit's current state, but this one needs to know how much CP the unit has *left* at the moment of entry — information `hex_entry_cost` doesn't have access to (it's written as a pure function of terrain/edge/movement class, deliberately, so it's easy to test in isolation). Implementing this properly means either threading remaining-CP into `hex_entry_cost`'s signature, or handling Enemy Minefields as a special case directly inside `resolve_move_order` rather than through the generic cost lookup. Left as unimplemented rather than guessed at, since minefields aren't placed on any map yet regardless.