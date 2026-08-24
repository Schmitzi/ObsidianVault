# Architecture: Why Four Crates, and Why an Embedded Server

## The problem this solves

CNA is a huge, stateful simulation that needs to run identically in three contexts: single-player, online multiplayer, and (eventually) vs-AI. If those three contexts each had their own code path for "what happens when a turn resolves," they would eventually disagree with each other — a bug fix applied to multiplayer wouldn't apply to single-player, or a replay recorded in one mode couldn't be played back correctly in another. The entire architecture exists to make that kind of drift *structurally impossible*, not just something we're careful about.

## The four crates, and what each one is not allowed to know

```
engine   — pure simulation. No networking, no rendering, no file I/O.
server   — owns authoritative state, runs engine, handles replay logs.
client   — egui UI, embeds a server instance for single-player.
shared   — network message types, used by both server and client.
```

**Why `engine` has zero I/O:** the moment `engine` reads a clock, reads a random OS entropy source, or touches the filesystem, `resolve_turn()` stops being a pure function. And `resolve_turn()` being pure — `state, orders, seed) -> new_state`, nothing else — is the single fact that everything else in this project depends on:

- **Replay** only works if replaying the same orders through the same seed always produces the same result. A hidden dependency on wall-clock time breaks this silently — replays would look right the first time and diverge later.
- **Server authority** only works if the server can be trusted to be the one source of truth. If `engine` had any hidden state, two "identical" engine instances (say, one on the authoritative server, one on a client doing local prediction) could disagree without either one being "wrong" in an obvious way.
- **Single-player = embedded server** (below) only works if the exact same `engine` code runs whether or not there's a network in the picture.

**Why `server` always runs, even offline:** the alternative — letting the client resolve turns locally for single-player, and only going through a real server for multiplayer — means two different code paths eventually resolve turns two different ways. That's not a hypothetical risk; it's what *always* happens over time in codebases that allow it, because the two paths get touched by different features at different times and nobody notices the drift until a player reports a bug that "only happens sometimes." Instead: single player = a `server` instance running in the same process, talking to the `client` over an in-memory channel instead of a socket. Same validation, same resolution, same replay logging, every time.

**Why `shared` exists separately from `engine`:** `engine` types are about *simulation* (a `Unit`, an `Order`). `shared` types are about the *network boundary* (an auth token, a lobby ID, the envelope wrapping an `OrderSet` as it crosses the wire). Mixing these would mean every time you touched networking, you'd risk touching simulation logic by accident, and vice versa. `shared` will need real content eventually — for now it's an empty crate because there's nothing to send over a network yet, but the seam is there so adding it later doesn't require restructuring anything.

## Why the replay system is "orders in, not video"

Trackmania-style: store *inputs*, not the resulting frames. For CNA this maps even more naturally than it does for a racing game — an `OrderSet` for a faction on a given turn is already tiny, structured data (a handful of `Order` enum values), not something that needs compression the way pixels do.

This works *only* because `resolve_turn()` is deterministic. That's the entire reason [fixed-point math](./01-fixed-point.md) and [seeded RNG](./04-resolution-sequence.md#determinism-and-rng) exist — they're not general-purpose "good practice," they are specifically the two things that would otherwise make replay silently wrong. See those docs for the mechanism.

The replay log stores, per turn: the seed, and each faction's `OrderSet`. To reconstruct any point in a campaign, replay every turn's orders through `resolve_turn()` from the start. For long campaigns this gets slow, which is why the design (not yet built) periodically snapshots full `GameState` every N turns — load the nearest snapshot, replay only the handful of turns since.

## Why WEGO got revised to "blind submission, sequential resolution"

Early in the design, WEGO (both sides submit orders blind, then resolve simultaneously) was chosen specifically because it fits async multiplayer — players don't need to be online at the same time, and nobody's move depends on seeing the opponent's move first.

But actually read the rulebook's Sequence of Play (5.2), and the real game is IGOUGO within an Operations Stage: Player A completes his *entire* Movement and Combat Phase — which can itself loop several times under Continual Movement (8.2) — before Player B gets his turn.

These aren't actually in conflict, once separated into two different questions:

1. **When do players decide their orders?** (submission) — can stay simultaneous/blind, preserving async play and fog-of-war.
2. **In what order does the engine apply them?** (resolution) — needed to be sequential, by initiative, to match the real game.

`resolve_turn()` takes an `initiative_order: &[FactionId]` and fully resolves one faction's order set before moving to the next. See [04-resolution-sequence.md](./04-resolution-sequence.md) for how that's implemented and what's still simplified about it.
