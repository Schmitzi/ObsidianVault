# IP-free Campaign for North Africa

The goal of this stupid project is to make a legally distinct copy of the "popular" 1979 board game "The Campaign for North Africa".

## What is CNA
Campaign for North Africa is built on a few key points:
- Logistics is the game, not a modifier on it. Water, fuel, ammo, and food are tracked separately per unit, and running out of any one of them is often more dangerous than the enemy.
- Absurd unit-level granularity (individual companies, gun batteries, and truck columns as pieces).
- A supply chain with real distance/capacity constraints. Stuff has to physically move from port → depot → front, and each step can bottleneck
- Environmental brutality (desert terrain, heat, sandstorms) actively working against both sides, not just window dressing.
- Asymmetric factions with wildly different logistics profiles (historically: Axis chronic fuel/supply shortage vs Allied material abundance but longer lines)

To make this more my own, I will be concidering a few design aspects:

- Different theater entirely (could dodge WWII altogether — Eastern Front, a fictional/sci-fi setting, modern-day, whatever), which sidesteps any resemblance question entirely.
- Let the player decide how much of the granularity the want to experience. The game can automate the bookkeeping that made CNA infamous (tracking pasta rations by hand) while keeping the tension.
- Custom faction asymmetry logic, unit roster, map, names, art.
- A "Log" file like trackmania. Use a file containing the actions to keep a true record of the game being played.

## Points

## Simple vs. Realistic

The easiest way to implement this will be to have one game engine with an "abstraction slider".

- **Realistic mode**: full simulation exposed. That means individual fuel/water/ammo pools per unit, manual convoy routing, weather affecting individual unit performance, attrition tracked to the man/vehicle.

- **Simple mode**: same simulation running underneath, but aggregated up a level. This way supply is tracked per _army group_ instead of per unit, the can game auto-routes convoys unless you intervene, weather becomes a periodic event/modifier rather than continuous friction.

## Turn-based Game Actions

The best way to handle turns is to use the "WEGO" system. Players submit orders blind, then a turn resolves simultaneously. This is what CNA and most serious wargames use, because it prevents "I-go-you-go" turns from letting whoever moves first dictate everything. It's also naturally async-friendly so players submit whenever they're ready within the window, no need to be online together.

## Server Authoritive State
- **Server owns resolution**: Clients never resolve turns locally and report results. They submit orders, and only the server-side engine runs resolve(state, orders) → new state. The client engine is just a predictor for UI responsiveness, always overridden by the server's result.
- Orders must be validated before acceptance, not just before resolution. If a player submits "route 500 fuel to unit X" but only has 200 available, that needs to be rejected/clamped at submission time, using server-side state. Never trust client-reported unit counts, fuel totals, or positions.
- RNG seeds are server-generated, not client-supplied, otherwise a player could submit a seed they know produces favorable combat rolls. Since you're recording "the event that took place" as you said, that means: server rolls using its own seed, resolves the event, and then writes the outcome (seed + result) into the replay log. Client never picks the seed.
- Single-player and vs-AI still route through the same authoritative resolver (even if "server" is just local logic running on-device) keeping the three modes (single-player/multiplayer/AI) sharing one resolution path, which also protects the replay format from splitting into two.

## Hiding Information
Since it's WEGO with fog-of-war (we want players not to see each other's supply routing), an authoritative server also has to enforce what each client is even allowed to see, not just what they're allowed to do. That's a second, separate validation layer from order-validation:

- State sent to Player A must be filtered to only what A's reconnaissance/intel would reveal, computed server-side from actual positions
- The replay log used for anti-cheat/audit can be the full omniscient state, while what gets shipped to each client each turn is a redacted view