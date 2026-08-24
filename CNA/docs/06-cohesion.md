# Cohesion

All units begin with `Cohesion Level` or `CL` of 0 with a possible maximum of 10. This level can stay 0 but will can also be `Positive` or `Negative`:

- *Negative Cohesion Level*: This unit is in `Disorganisation`, that means it has probably been exerting itself beyond its usual "limit" by moving too much or attacking to often.
- *Positive Cohesion Level*: This unit is in `Reorganisation`, meaning it has been winning battles.

The *CL* of a unit directly impacts its `Morale Level` or `ML` at the instant of combat, which then affects its performance in said battle. The units CL is kept track of individually. This is wont be an issue because most units under the same Parent Unit will have the same level.

If you have more than one unit ( or individual combat counter) in a Close Assault, and the indivudual units have different CL's, then the CL of the largest (in terms of division, brigade, etc) unit prevails determined by [Stacking Points](07-stacking.md) or SP's.

If you are determining adjustments to Morale through CL and have more than one "largest unit", then use this fomula.

$$ newCL=\frac{\sum_{i=1}^{n}CL_{i}}{n} $$

So say three Brigades are assulting an emeny unit. The CL of the Brigades is -4, -1 and +3 respectively. SO we add them together (-4 + (-1) + 3 = -2) then divide that by three (-2 / 3 = -1), rounding to the nearest whole number. So for this battle the CL is -1.

## Exceeding the Cohesion Point Allowance

If and when a unit excees its given CPA within the Ooperations Stage, it will earn for itself `Disorganisation Points` or `DP`. These are credited immediatley and not at the end of the Operations Stage. For every *CP* that a unit expends over is CPA, it earns one DP. That means a unit with a CP of 15 that uses 18 CP's in an Operations Stage earns itself three DP's

The DP's also decrease the CL of a unit. So if a unit had CL of -1, then earns three more DP's, it will now have a CL of -4. The DP's also accumulate from Segment to Segment and may only be negated by earning `Reorganisational Points` or `RP`.

**Sequencing note (engine-relevant):** the rulebook's own worked example (a unit that overspends CPA in Movement, then immediately Close Assaults in the same Segment) makes clear the CPA-overspend DP is applied *before* that unit's own combat resolves — the lowered CL is what the *upcoming* fight actually uses for Morale, not something calculated afterward. This means CPA-driven Cohesion changes can't be deferred to a separate end-of-stage Attrition pass; they need to apply at the moment the overspend happens (i.e. inline in Movement, where `ResolvedEvent::CpaExceeded` already fires), before Combat resolves for that unit.


## How to earn Disorganistation Points
- Suffering losses of 30% or greater in a single Close Assault. This applies to the *Parent Formation as well as all units in that formation* — not just the unit that fought — and earns a flat 3 DP's. (See also Case 15.29b, not yet read since Combat isn't implemented.)
- Moving unit too far for its CPA

## Increasing Cohesion Level

A units CL can be increased by earning RP's. The units maximum CL is 10. The RP's earned are added to a unit's CL as the DP's are, e.g CL = -3, earn 5 points, new CL = +2. Just like DP's, these are applied immediately.

RP's can be earned in a number of ways:
- If during an Operations Stage, the unit uses none of its CP's (except for a unit undergoing Training of conducting Training), then it earns `5 RP's`. However, these RP cannot be added if a units CP is positive:
```
CL = -3, earns 5 RP, new CL 0 (not +2)
```
  **This cap only applies to this specific RP source.** It does not apply to RP earned any other way — including the general case above (`CL = -3, earn 5 points, new CL = +2`) or the Close Assault bonus below, both of which can push CL past 0 uncapped.
- During a `Close Assault`, if a defending unit vacates its `hex` completely as a direct result of said Close Assault (not `Reaction` or `Retreat Before Battle`), then if it's victorious, that unit earn `3 RP's`.

## Very Disorganised Units

When a unit has a CL of -26 (or worse), it may not longer:

- Move
- Attack
- Defend

If an enemy combat unit moves adjacent to it, it immediately surrenders, regardless of the size of the enemy unit. Disorganised units are still allowed to:

- Refuel
- Consume Stores and Water *(this specific line references the fuller Logistics Game's Stores/Water tracking — per 32.11, the abstract Land Game we're building first explicitly excludes Water/Stores, so this isn't something to implement yet)*

The CL of the unit may drop below -26 if it performs a single action that would jump past this level such as an attack