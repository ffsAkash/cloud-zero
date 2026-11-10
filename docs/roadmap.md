# Implementation roadmap

This roadmap assumes standard chess, C++20, classical evaluation, and one search worker. Follow the dependency order. The suggested weeks correspond to the [study plan](study-plan.md), but completion depends on evidence rather than elapsed time.

## Phase 0 — Tools and project foundations

**Suggested time:** week 1. **Milestone:** M0.

Read the [C++ development guide](cpp-development.md), the [CMake tutorial](https://cmake.org/cmake/help/latest/guide/tutorial/index.html), and the [AddressSanitizer documentation](https://clang.llvm.org/docs/AddressSanitizer.html).

### Work sequence

1. Create a minimal executable and a separate test executable.
2. Choose a test framework or a small explicit test runner.
3. Configure C++20 and compiler warnings.
4. Separate debug and release build directories.
5. Enable memory and undefined-behavior diagnostics in a development configuration.
6. Learn how to set a breakpoint, inspect a stack frame, and evaluate an expression.
7. Write down the build and test commands that actually work on your machine.

### Exercises

- Explain the difference between a compiler error, linker error, failed assertion, and sanitizer report.
- Explain why an optimized build can behave differently when a program has undefined behavior.
- Find out whether your chosen test assertions remain active in release builds.

### Acceptance gate

A fresh build directory can produce both executables. A deliberate test failure is detected. You can inspect the failed calculation with a debugger. No engine code is required.

## Phase 1 — Position representation and FEN

**Suggested time:** week 2. **Milestone:** M1.

Read [Board Representation](https://chessprogramming.org/Board_Representation), [FEN](https://chessprogramming.org/Forsyth-Edwards_Notation), and [architecture](architecture.md).

### Work sequence

1. Choose and document a square mapping.
2. Define color, piece type, piece, and square concepts.
3. Represent a board with 64 squares.
4. Add side to move, castling rights, en passant target, and move counters.
5. Implement square/coordinate conversion.
6. Print the board in a readable orientation with coordinates.
7. Implement FEN parsing and export.
8. Reject malformed input without leaving partially modified live state.

Decide whether parsing accepts syntactically valid but impossible positions. Keep that policy separate from basic format validation.

### Exercises

- Draw your index mapping and label every corner.
- Identify two positions with the same pieces but different castling possibilities.
- Explain why FEN cannot establish whether a position has occurred three times.
- List the six FEN fields without looking them up.

### Acceptance gate

All square conversions round-trip. A variety of positions retain their state through parse/export/parse. Invalid input produces useful errors. Do not require exact string equality if your exporter intentionally canonicalizes equivalent input.

## Phase 2 — Attacks and ordinary pseudo-legal moves

**Suggested time:** weeks 3–4. **Partial milestone:** M2.

Read [Move Generation](https://chessprogramming.org/Move_Generation) and the movement rules in [FIDE article 3](https://handbook.fide.com/chapter/E012023).

### Work sequence

1. Implement a square-attacked query independent of legal move generation.
2. Add knight and king attacks.
3. Add pawn attacks separately from pawn pushes.
4. Add sliding attacks with explicit blocking rules.
5. Generate ordinary piece moves.
6. Add pawn single pushes, double pushes, and ordinary captures.
7. Represent moves in a readable form suitable for test diagnostics.

Use coordinate bounds to prevent file wrapping. A numeric destination inside 0–63 is insufficient to prove a move stayed on the intended file or diagonal.

### Exercises

- Count a knight's attacks on a1, a4, and d4.
- Explain why a pinned piece can attack a square even though moving there would expose its own king.
- Explain why the opponent's legal move generator is an awkward foundation for attack detection.
- Place a blocker on each square of a rook ray and predict the resulting attacks.

### Acceptance gate

Every ordinary piece has empty-board and blocker tests. Pawn pushes and attacks are distinguishable. Friendly captures and enemy-king captures are excluded from candidate moves.

## Phase 3 — Legal moves and special moves

**Suggested time:** weeks 5–6. **Milestone:** M2.

Read [Make Move](https://chessprogramming.org/Make_Move), [Unmake Move](https://chessprogramming.org/Unmake_Move), and [rules and edge cases](rules-and-edge-cases.md).

### Work sequence

1. Apply an ordinary move to a copied position.
2. Filter candidates by checking the moving side's king after the move.
3. Add all four promotion choices, including capture promotions.
4. Add en passant and its special captured-pawn square.
5. Add castling with explicit start, transit, destination, occupancy, and rights checks.
6. Update counters and rights for every relevant move.
7. Introduce an undo record and implement reversible in-place changes.
8. Compare copied-child and make/unmake results during testing.

A copied position is a useful initial reference. You may migrate earlier to make/unmake if you can still isolate state-restoration failures.

### Exercises

- List everything that must be restored after undoing castling.
- Explain why an en passant capture can expose an attack along a rank.
- Explain why moving a rook away and back does not restore castling rights.
- Explain why inspecting only the castled final position is insufficient.

### Acceptance gate

All special-move cases in the test guide pass. For every legal move in each fixture, make/unmake restores every meaningful field. Random legal sequences can be completely undone.

## Phase 4 — Move-generation verification

**Suggested time:** weeks 7–8. **Milestone:** M3.

Read [testing](testing.md) and [Perft Results](https://chessprogramming.org/Perft_Results).

### Work sequence

1. Implement a simple perft that uses legal moves.
2. Implement root divide output.
3. Match the published counts for the initial position.
4. Match Kiwipete and the remaining CPW reference positions.
5. Compare sorted legal move sets against an independent implementation.
6. Generate reproducible random legal positions and compare again.
7. Preserve the smallest failure from every bug as a regression test.

Keep the first perft implementation deliberately simple. Introduce bulk counting, caching, or other shortcuts only after verifying them against it.

### Exercises

- Demonstrate how two move-generation errors could cancel in the total count.
- Find the first differing branch in an intentionally damaged implementation.
- Explain why perft should not stop at a fifty-move draw.

### Acceptance gate

All six standard reference positions pass through depth four, unless a documented runtime constraint requires a temporary smaller gate. Add initial-position depth five as a useful extended check. Independent move-set comparisons and state invariants also pass. These checks are strong evidence, not a mathematical proof of correctness.

## Phase 5 — Results and game history

**Suggested time:** week 9. **Milestone:** M4.

Read FIDE articles 5 and 9 and the draw discussion in [rules and edge cases](rules-and-edge-cases.md).

### Work sequence

1. Recognize checkmate and stalemate.
2. Maintain halfmove and fullmove counters.
3. Store enough history for repetition checks.
4. Define repetition identity, including legal en passant possibilities.
5. Detect a conservative set of dead positions.
6. Document claimable-draw, automatic-draw, and search-cycle policies separately.
7. Establish precedence when mate and a move-count threshold coincide.

### Exercises

- Construct a repeated board layout that is not a repeated position because castling rights changed.
- Explain why inability to force mate does not imply a dead position.
- Explain what repetition information can be recovered from a standalone FEN.

### Acceptance gate

Terminal-state fixtures pass. Repetition tests use actual histories. The documented policies distinguish game adjudication from search shortcuts. Advanced dead-position recognition may remain a stated limitation.

## Phase 6 — Static evaluation and exhaustive search

**Suggested time:** weeks 10–11. **Milestone:** M5.

Read [Evaluation](https://chessprogramming.org/Evaluation) and [Negamax](https://chessprogramming.org/Negamax).

### Work sequence

1. Select a score perspective and safe integer range.
2. Implement material evaluation.
3. Test evaluation with both sides to move.
4. Search tiny artificial trees with known answers.
5. Implement shallow exhaustive negamax on chess positions.
6. Recognize terminal states before ordinary depth-limit evaluation.
7. Introduce consistent mate-distance scores.
8. Add a way to inspect root moves and their scores.

### Exercises

- Manually solve a depth-three tree containing both positive and negative leaves.
- Explain why initializing the best score to zero is wrong when all legal moves lose.
- Trace the sign of a score through three recursive calls.
- Show how mate-distance scoring chooses between two forced wins.

### Acceptance gate

Toy-tree answers match. The engine finds simple captures and mates at the required depth. Black-to-move tests pass. Search leaves the root state unchanged.

## Phase 7 — Alpha-beta and quiescence

**Suggested time:** weeks 12–13. **Milestone:** M6.

Read [Alpha-Beta](https://chessprogramming.org/Alpha-Beta), [Quiescence Search](https://chessprogramming.org/Quiescence_Search), and [search and evaluation](search-and-evaluation.md).

### Work sequence

1. Add alpha-beta without selective pruning.
2. Compare full-window results to exhaustive negamax.
3. Test bound behavior on narrow windows.
4. Add simple capture ordering.
5. Add quiescence with captures and promotions outside check.
6. Search all legal evasions when in check and disallow stand pat there.
7. Preserve mate and stalemate detection at the horizon.
8. Track main-search and quiescence node counts separately.

### Acceptance gate

Alpha-beta agrees with its exhaustive equivalent under identical leaf semantics. Tests cover poisoned captures, quiet evasions, promotions, and terminal horizon positions. Guard against recursion overflow without silently treating an in-check position as an ordinary static leaf.

## Phase 8 — Iterative deepening, UCI, and time limits

**Suggested time:** weeks 14–15. **Milestone:** M7.

Read [Iterative Deepening](https://chessprogramming.org/Iterative_Deepening) and the specification linked on [Shredder's UCI page](https://www.shredderchess.com/chess-features/uci-universal-chess-interface.html).

### Work sequence

1. Search successive depths and retain completed results.
2. Establish a legal fallback before substantial search begins.
3. Add node or stop checks at a controlled interval.
4. Implement UCI identification and readiness.
5. Parse positions and move lists.
6. Implement fixed-depth and fixed-time search.
7. Add clock and increment handling conservatively.
8. Keep input responsive during search.
9. Handle stop, quit, end-of-input, and repeated games.
10. Play complete games and inspect protocol logs.

### Acceptance gate

The engine plays legal complete games, stops promptly, produces one result per search, and remains usable after a stopped search. A protocol checker and transcript tests supplement manual GUI play.

## Phase 9 — Position hashing and cached search

**Suggested time:** weeks 16–17. **Milestone:** M8.

Read [Zobrist Hashing](https://chessprogramming.org/Zobrist_Hashing) and [Transposition Table](https://chessprogramming.org/Transposition_Table).

### Work sequence

1. Generate deterministic random keys.
2. Implement complete hash recomputation.
3. Implement incremental updates and compare both methods.
4. Add table entries with keys, depth, scores, bounds, and moves.
5. Implement a simple replacement policy.
6. Normalize mate-distance scores on storage and retrieval.
7. Test index collisions with deliberately small tables.
8. Account for history-sensitive results and move-count draws.

### Acceptance gate

Hash recomputation checks pass after every tested transition. Controlled searches agree with caching disabled. Bound, depth, collision, and mate-normalization tests pass. Draw-related reuse restrictions are documented.

## Phase 10 — Experimental improvement

**Suggested time:** week 18 onward. **Milestone:** M9.

Possible tracks include richer evaluation, stronger ordering, PVS, aspiration windows, bitboards, static exchange evaluation, selective search, tuning, and eventually parallelism or neural evaluation.

For each experiment:

1. State the expected benefit.
2. Identify likely failure modes.
3. Save a baseline version.
4. Make one meaningful change.
5. Run relevant correctness checks.
6. Measure speed or playing strength under controlled conditions.
7. Keep, revise, or revert based on the evidence.

Do not introduce multiple pruning techniques together. Their interactions make failures difficult to attribute.

### Acceptance gate

Every retained improvement has a reproducible comparison and known limitations. A higher nodes-per-second count alone does not establish stronger play.
