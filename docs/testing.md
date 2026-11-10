# Testing and verification

Testing should establish specific claims about behavior. A plausible move, a high node rate, and a few won games answer different questions. Build a collection of small tests before relying on large game runs.

## Verification layers

| Layer | Question answered | Examples |
| --- | --- | --- |
| Unit tests | Does one operation obey its contract? | Square conversion, FEN fields, pawn attacks |
| Transition tests | Does a move update all required state? | Captures, rights, counters, promotion |
| Property tests | Does an invariant hold across many inputs? | Make/unmake equality, hash recomputation |
| Perft | Does the legal move tree match known counts? | Published CPW positions |
| Differential tests | Does an independent implementation agree? | Exact legal move sets and child states |
| Search tests | Does decision logic return the expected result? | Toy trees, mate positions, score bounds |
| Protocol tests | Does the executable behave as a UCI process? | Readiness, stop, repeated searches |
| Match tests | Does a change improve practical play? | Paired engine games under fixed conditions |

Each layer catches errors the others can miss. Perft does not establish correct evaluation or repetition adjudication.

## Test data records

Store expected results with enough context to audit them:

```text
Identifier:
Purpose:
FEN:
Prior move history, if needed:
Operation:
Expected moves/state/count/score:
Source or hand-derived reasoning:
Independent verification:
Bug originally caught:
```

Keep fixtures readable. If a position was minimized from a larger failure, retain both the small regression case and the original reproduction when useful.

## Position and notation tests

Test every coordinate in both directions, all piece symbols, both colors, castling-right subsets, en passant fields, and counter values.

Malformed FEN cases should include missing fields, too many ranks, rank widths other than eight, unknown piece symbols, invalid side-to-move tokens, duplicate or invalid rights symbols according to your policy, and invalid numeric counters.

Separate structural validity from legal-position validity. For example, a syntactically correct board with no kings may be rejected by a legal-position validator without being a FEN tokenization error.

If serialization normalizes a field, compare semantic state after re-parsing rather than requiring the original text verbatim. Document the en passant normalization convention.

## Attack and move tests

For each piece and color, cover center, edge, corner, friendly blockers, enemy blockers, and king-safety interactions.

Compare **sets of complete moves**, including promotion type. A move-count-only assertion can pass with one missing and one extra move. Also assert there are no duplicate generated moves.

Attack tests should not depend on the correctness of legal move generation. Otherwise both can share an error and agree falsely.

## State transition properties

For every legal move in a fixture:

1. Save a logical snapshot of the parent.
2. Make the move.
3. Check child invariants and expected changed fields.
4. Compare the child with an independent or copied-state result where available.
5. Undo.
6. Compare every meaningful parent field.

Include piece placement, side to move, rights, en passant, counters, king locations, hashes, incremental evaluation, and history indices when present.

Do not use only FEN equality: caches or history can be wrong while serialized state looks correct. Do not use only hash equality: hashes omit some fields and can collide. Avoid raw struct-memory comparisons because of padding.

For long-sequence tests, save the seed and complete move list. Undo every move and verify both the final original state and intermediate snapshots.

## Perft semantics

At depth zero, count one leaf. At positive depth, add the counts from every legal child. A terminal position reached before the target depth contributes zero leaves at the requested depth.

Perft counts legal paths, not unique positions. Ignore repetition, dead-position, and move-count draw termination. Exclude evaluation, alpha-beta, and selective search.

Begin without hashing or bulk-count shortcuts. Those optimizations can hide transition bugs if used before the reference path is trusted.

### Reference fixtures

The following FENs and counts come from [CPW Perft Results](https://chessprogramming.org/Perft_Results). Copy each FEN exactly; similar-looking variants may have different results.

#### P1 — Initial position

```text
rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1
```

#### P2 — Kiwipete

```text
r3k2r/p1ppqpb1/bn2pnp1/3PN3/1p2P3/2N2Q1p/PPPBBPPP/R3K2R w KQkq - 0 1
```

#### P3 — CPW position 3

```text
8/2p5/3p4/KP5r/1R3p1k/8/4P1P1/8 w - - 0 1
```

#### P4 — CPW position 4

```text
r3k2r/Pppp1ppp/1b3nbN/nP6/BBP1P3/q4N2/Pp1P2PP/R2Q1RK1 w kq - 0 1
```

#### P5 — CPW position 5

```text
rnbq1k1r/pp1Pbppp/2p5/8/2B5/8/PPP1NnPP/RNBQK2R w KQ - 1 8
```

#### P6 — CPW position 6

```text
r4rk1/1pp1qppp/p1np1n2/2b1p1B1/2B1P1b1/P1NP1N2/1PP1QPPP/R4RK1 w - - 0 10
```

| Fixture | Depth 1 | Depth 2 | Depth 3 | Depth 4 | Depth 5 |
| --- | ---: | ---: | ---: | ---: | ---: |
| P1 | 20 | 400 | 8,902 | 197,281 | 4,865,609 |
| P2 | 48 | 2,039 | 97,862 | 4,085,603 | 193,690,690 |
| P3 | 14 | 191 | 2,812 | 43,238 | 674,624 |
| P4 | 6 | 264 | 9,467 | 422,333 | 15,833,292 |
| P5 | 44 | 1,486 | 62,379 | 2,103,487 | 89,941,194 |
| P6 | 46 | 2,079 | 89,890 | 3,894,594 | 164,075,551 |

Depth zero is one for every fixture. Use a 64-bit count type and check the range before attempting larger depths. Depth five for every position need not be part of every development run.

### Divide workflow

Divide prints one subtotal per root move. For a mismatch:

1. Compare exact root move sets.
2. Select one move with a differing subtotal.
3. Make that move in both implementations.
4. Compare the resulting complete state.
5. Re-run divide at one lower depth.
6. Repeat until an immediate discrepancy remains.

Record the move prefix along with the child FEN. FEN by itself can lose history relevant to other classes of bug.

## Small regression positions

These are hand-reasoned assertion fixtures. They are independent exercises rather than published perft totals. Verify the reasoning when you implement them.

### R1 — Checkmate

```text
7k/6Q1/5K2/8/8/8/8/8 b - - 0 1
```

Black's king on h8 is checked by the queen on g7. The queen covers escapes and is protected by the white king. Expect checkmate and an empty legal move set. At perft depth zero expect one; at positive depth expect zero.

### R2 — Stalemate

```text
7k/5Q2/6K1/8/8/8/8/8 b - - 0 1
```

Black is not in check, but all king escapes are covered. Expect stalemate, including if this position is reached at the normal search horizon.

### R3 — Quiet promotion choices

```text
7k/P7/8/8/8/8/8/7K w - - 0 1
```

The legal move set includes `a7a8q`, `a7a8r`, `a7a8b`, and `a7a8n`. These are not the only legal moves because White's king can move. Make each promotion, verify the resulting piece, and undo it.

### R4 — En passant exposes the king

```text
k7/8/8/r4pPK/8/8/8/8 w - f6 0 1
```

Exclude `g5f6`. Removing the white pawn from g5 and black pawn from f5 opens the a5–h5 rook line onto White's king. The raw en passant target therefore does not establish a legal en passant possibility for repetition identity.

### R5 — Both castling directions available

```text
r3k2r/8/8/8/8/8/8/R3K2R w KQkq - 0 1
```

Include `e1g1` and `e1c1`. Verify rook movement, rights removal, counters, and complete undo. Derive variants with one missing right, one blocker, or one attacked king-path square.

### R6 — Quiet check evasion at the horizon

```text
4r2k/8/8/8/8/8/8/4K3 w - - 0 1
```

White is checked along the e-file. Legal king escapes exist, and they are quiet moves. A captures-only quiescence generator would miss them. Assert that quiescence searches legal evasions instead of accepting stand pat or reporting mate.

## History and adjudication tests

Use actual move sequences for repetition. A useful start-position sequence is:

```text
g1f3 g8f6 f3g1 f6g8
```

This returns to the original repetition identity. Repeating the cycle produces additional occurrences. Counters differ, so do not compare full FEN strings as repetition keys.

Add tests for changed castling rights, relevant versus irrelevant en passant, 99/100 and 149/150 halfmove boundaries, pawn/capture resets, and mate on the move that reaches the automatic threshold. Test your stated claim policy separately from strict automatic adjudication.

## Differential testing

Use an independent implementation such as [Disservin's C++ chess library](https://github.com/Disservin/chess-library) as a test-only oracle.

For each legal position:

1. Normalize both move sets to UCI coordinate notation.
2. Sort and compare sets and duplicate counts.
3. Apply corresponding moves and compare child state.
4. Compare checks and terminal status where semantics agree.

Account for FEN en passant serialization differences before declaring a mismatch. An oracle is evidence, not infallibility; investigate disagreements with small positions and rules reasoning.

Generate random positions through legal play rather than arbitrary piece placement. Retain the seed, initial FEN, full move list, and oracle version. Avoid stopping all generation at simple draws if that prevents exercising useful transition cases; distinguish the move-generation test mode from game adjudication.

## Search-specific tests

- Solve toy trees manually before using chess positions.
- Compare exhaustive search with full-window alpha-beta under identical leaf semantics.
- Permute move order; scores should agree, though equal best moves can differ.
- Test narrow-window bounds rather than requiring exact scores from every call.
- Verify mate distances and Black-to-move sign handling.
- Replay every reported principal variation legally.
- Ensure cancellation does not change the root or cache incomplete scores.
- Compare caching enabled/disabled on controlled histories.

Tactical test expectations must account for search depth and available information. A move that is objectively best may be invisible at shallow depth.

## Protocol and lifecycle tests

Drive the executable as a subprocess and inspect complete transcripts. Test identification, readiness, position setup, depth/time limits, stop, quit, empty input, and repeated games.

Use sensible timing tolerances for process scheduling. Establish a maximum stop latency target, measure it, and log failures. Verify output lines remain intact if multiple threads can produce messages.

[Fastchess](https://github.com/Disservin/fastchess) provides match tooling and a UCI compliance checker; use its current documentation for invocation details.

## Suggested suite tiers

| Tier | Contents | Typical use |
| --- | --- | --- |
| Fast | Unit tests, small fixtures, shallow perft, toy search trees | Every relevant edit |
| Integration | All reference perft to depth four, seeded comparisons, protocol transcripts | Before milestone completion |
| Extended | Deeper perft, larger random campaigns, sanitizer runs | Before major releases or shared-state changes |
| Experimental | Benchmarks, tactics, paired matches | Evaluating a specific improvement |

Run tests appropriate to the change. Broaden checks when new failures, shared-state edits, or unresolved concerns justify it. Repeatedly running expensive matches after a small documentation edit provides no useful evidence.
