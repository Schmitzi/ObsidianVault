# The Hex Map: From Abstract Graph to Real Terrain

## Where this started, and why it changed

The very first version of `state.rs` modeled position as a `NodeId`. Units sat "at" a depot/port/city, and the map was a graph of named locations connected by edges with a distance.

Then Rules 8.0 (Land Movement) turned out to describe something a graph literally cannot represent: a unit moving from A to B accumulates cost **hex by hex**, and the cost of each individual hex depends on that hex's specific terrain (8.31, 8.37). A graph edge has one weight; it can't say "the first 3km of this trip cost 1 CP each, then this next km cost 8 CP because it's an escarpment." So the map had to become an actual hex grid.

This is worth internalizing as a pattern for the rest of this project: model based on what the rules actually say, not on what's easiest to write first. The graph version wasn't *wrong* exactly but it was modeling a different, simpler game than the one being built. Re-reading rules before committing to a data structure is going to keep happening, and that's fine; it's cheaper to redo a `state.rs` now than after five more systems depend on the wrong shape.

## Axial coordinates: what `HexCoord { q, r }` means

Hex grids don't have a single "obvious" coordinate system the way square grids do (`x, y`). We used **axial coordinates**, which is one of the two most common schemes (the other being "offset," which uses row/column with a parity correction). Axial was chosen because:

- **Adjacency is uniform.** All six neighbors of `(q, r)` are found by adding one of six fixed `(dq, dr)` pairs — no "even row vs odd row" special-casing that offset coordinates need.
- **It matches how `neighbours()` is implemented** — six constant direction vectors, applied directly:

  ```rust
  const DIRS: [(i32, i32); 6] = [(1, 0), (1, -1), (0, -1), (-1, 0), (-1, 1), (0, 1)];
  ```

If you're not familiar with axial hex coordinates, the standard reference is Red Blob Games' hex grid guide since every hex-grid concept in this codebase (`is_adjacent_to`, pathing, distance) assumes that mental model.

One thing this codebase does *not* yet do: map `HexCoord` values to the actual CNA map's real hex-numbering scheme (the rulebook uses labels like "A2109" or "Hex 3613"). Right now `HexCoord` is an abstract axial coordinate with no connection to the physical game-map layout. That mapping — loading the real map's a few thousand hexes with correct terrain and adjacency — is a distinct, later task from the movement *mechanics* we're building now.

## Why terrain got split into two types

```rust
pub enum Terrain { OpenDesert, SaltMarsh, Oasis, MajorCity, Port, Coastal }
pub enum HexsideFeature { Road, Track, Railroad, Wadi, Ridge, Escarpment { high_side } }
```

Rule 8.35 draws this distinction directly: some terrain *fills* a hex (you're standing in open desert), while other terrain only matters when *crossing between two specific hexes* (a road, an escarpment). A single `Terrain` enum covering both would force awkward questions like "which hex does the escarpment belong to?" — when the honest answer is neither; it belongs to the *boundary* between two hexes.

Modeling this as `Map.edges: HashMap<(HexCoord, HexCoord), HexsideFeature>` means: a hex has exactly one terrain (simple lookup), and the cost of moving between two specific hexes is a separate, explicit lookup keyed by the pair.

### Why edges are canonicalized

```rust
pub fn edge_key(a: HexCoord, b: HexCoord) -> (HexCoord, HexCoord) {
    if (a.q, a.r) <= (b.q, b.r) { (a, b) } else { (b, a) }
}
```

The edge between hex A and hex B is the same physical hexside whether you're moving A→B or B→A. Without canonicalizing, you'd either have to store every edge twice (A→B *and* B→A, and remember to keep both in sync when either changes) or risk a lookup miss depending on which direction you query from. Sorting the pair before using it as a key means there's exactly one entry per hexside, looked up the same way regardless of travel direction.

### Why `Escarpment` carries `high_side`, but `Ridge` doesn't

Rule 8.35 explicitly says escarpments have a direction — moving up costs differently than moving down — while ridges are symmetric ("two-sided slopes," same cost either way). `Escarpment { high_side: HexCoord }` records *which* of the two hexes is the "up" side, so `hex_entry_cost()` can compare `high_side == from` to determine whether a given crossing is ascending or descending. `Ridge` needs no such field because the rule says direction doesn't matter for it — the type signature reflects a fact about the game rule, not just a modeling preference.

## What's still a placeholder here

The actual Terrain Effects Chart (8.37) — the numeric CP-cost table — could not be read from the uploaded PDF; that page's OCR came out as garbled characters. `hex_entry_cost()` (in `resolve.rs`) currently uses the two real numbers directly quoted in the rule *text* (1/4 CP for roads, +8 CP down an escarpment via track) and clearly-labeled placeholders for everything else. See [03-cpa-and-movement.md](./03-cpa-and-movement.md#the-terrain-effects-chart-gap) for what needs to happen once that chart is transcribed.
