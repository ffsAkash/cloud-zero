# Study plan

This is an approximately 18-week route to a tested classical engine, followed by open-ended improvement. It assumes basic programming knowledge and 6–10 focused hours per week. If C++ is new, allow additional foundation weeks. Dates are estimates; the [roadmap acceptance gates](roadmap.md) determine progress.

## Weekly structure

Use four sessions when possible:

| Session | Suggested duration | Outcome |
| --- | --- | --- |
| Reading and prediction | 60–90 minutes | Explain the feature and predict difficult cases |
| Design and implementation | 2–3 hours | A small working increment |
| Tests and investigation | 2–3 hours | Evidence and reduced failures |
| Review and notes | 45–90 minutes | A clean checkpoint and a next step |

These proportions are flexible. During move generation, testing may take more time than implementation. A session spent reducing one subtle bug is productive progress.

## Before every implementation session

Write three sentences:

1. The behavior I am adding is …
2. The smallest example that demonstrates it is …
3. I will know it works when …

Then list the input state, changed state, and unchanged invariants. This makes accidental scope growth easier to notice.

## Week-by-week plan

### Week 1 — Become comfortable with the tools

**Read:** selected LearnCpp material on compilation, functions, objects, references, arrays, and debugging; the CMake tutorial introduction; sanitizer documentation.

**Build:** your own executable and test runner, debug/release configurations, and a repeatable test command.

**Practice:** intentionally introduce an out-of-bounds access in a disposable exercise, inspect the diagnostic, and remove the faulty exercise afterward. Learn how to view a call stack.

**Explain from memory:** the difference between stack lifetime, heap lifetime, and a dangling reference.

**Finish when:** the M0 acceptance gate passes.

### Week 2 — Positions and notation

**Read:** CPW Board Representation and FEN.

**Build:** square mapping, piece types, position state, display, FEN import/export.

**Practice:** draw five positions on paper and encode them as FEN. Decode them without using the display, then compare.

**Explain from memory:** why piece placement alone is not a complete chess position.

**Finish when:** every coordinate conversion and FEN-state round-trip passes.

### Week 3 — Attack detection

**Read:** piece movement rules and CPW Move Generation.

**Build:** pawn, knight, king, and sliding attack queries.

**Practice:** make one minimal blocker fixture per sliding direction. Include board edges and pinned attackers.

**Explain from memory:** attacks versus moves, and pawn attacks versus pawn pushes.

**Finish when:** attack sets are predictable and thoroughly tested.

### Week 4 — Ordinary moves

**Build:** ordinary pseudo-legal moves, captures, pawn pushes, and readable move output.

**Practice:** compute complete move lists by hand on sparse positions. Test both colors rather than assuming symmetry.

**Explain from memory:** how a numeric index can remain in bounds while wrapping onto the wrong file.

**Finish when:** ordinary move tests pass with no file-wrap errors.

### Week 5 — Legality and promotions

**Read:** CPW Make Move and Unmake Move.

**Build:** copied child positions, king-safety filtering, promotion choices, and check-evasion cases.

**Practice:** create pinned-piece and discovered-attack fixtures. Inspect the board after each candidate move.

**Explain from memory:** which side's king must be checked after side-to-move changes.

**Finish when:** ordinary legal moves and all promotions pass targeted tests.

### Week 6 — Castling, en passant, and undo

**Read:** the special-move sections of the rules guide.

**Build:** both special moves, castling-right updates, counters, and make/unmake.

**Practice:** compare every child against your copied-position baseline. Undo long deterministic legal sequences.

**Explain from memory:** everything that must be restored after an en passant capture or castling move.

**Finish when:** M2 passes.

### Week 7 — Perft and divide

**Read:** CPW Perft Results and the perft section of the testing guide.

**Build:** perft and root divide.

**Practice:** start with shallow depths and follow discrepancies to their first incorrect move. Turn each discovery into a fixture.

**Explain from memory:** why perft counts paths, not unique positions.

**Finish when:** the initial position and Kiwipete pass practical depths.

### Week 8 — Independent verification

**Build:** comparisons against an independent move generator and seeded random sequences.

**Practice:** compare exact move sets, resulting position state, and complete undo. Include all CPW reference positions.

**Explain from memory:** why equal node counts do not imply equal move sets.

**Finish when:** M3 passes.

### Week 9 — Results and history

**Read:** FIDE articles 5 and 9.

**Build:** mate, stalemate, repetition history, move-count state, and a conservative dead-position recognizer.

**Practice:** test histories that return pieces to their original squares while losing castling rights. Test en passant identity with and without a legal capture.

**Explain from memory:** claimable versus automatic draws, and why a FEN is insufficient for repetition adjudication.

**Finish when:** M4 passes with limitations documented.

### Week 10 — Evaluation

**Read:** CPW Evaluation.

**Build:** material scoring, score-perspective tests, and an inspectable evaluation breakdown.

**Practice:** remove a piece and predict the score difference. Swap colors with a corresponding board transformation and predict the result.

**Explain from memory:** score perspective and why mate needs a separate score range.

**Finish when:** static scores behave exactly as intended.

### Week 11 — Exhaustive search

**Read:** CPW Negamax.

**Build:** toy-tree search, shallow chess negamax, terminal handling, and root score reporting.

**Practice:** trace a three-ply search manually. Use losing positions to catch incorrect zero initialization.

**Explain from memory:** score negation at each edge and the difference between depth and root-relative ply.

**Finish when:** M5 passes.

### Week 12 — Alpha-beta

**Read:** CPW Alpha-Beta.

**Build:** alpha-beta with the same leaves as exhaustive search.

**Practice:** permute move order and compare full-window scores. Test narrow windows and deliberately triggered cutoffs.

**Explain from memory:** what a cutoff proves and what it does not prove about the exact score.

**Finish when:** reference comparisons pass.

### Week 13 — Quiescence

**Read:** CPW Quiescence Search.

**Build:** capture ordering, tactical continuation search, legal evasions in check, and horizon terminal handling.

**Practice:** poisoned captures, quiet promotions, quiet check evasions, and stalemates at depth zero.

**Explain from memory:** stand pat and why it is forbidden while in check.

**Finish when:** M6 passes.

### Week 14 — Iterative deepening and protocol basics

**Read:** CPW Iterative Deepening and the UCI specification.

**Build:** completed-iteration tracking, identification, readiness, position parsing, and fixed-depth search.

**Practice:** record and inspect complete input/output transcripts.

**Explain from memory:** why an interrupted result needs a completion status distinct from an ordinary score.

**Finish when:** fixed-depth searches work through an external UCI client.

### Week 15 — Time and lifecycle

**Build:** time budgets, responsive stop handling, shutdown, and repeated-game behavior.

**Practice:** tiny budgets, stop immediately after go, no-legal-move roots, and quitting during search.

**Explain from memory:** which thread owns each mutable object if you use an input thread and search worker.

**Finish when:** M7 passes and complete games run reliably.

### Week 16 — Hashing

**Read:** CPW Zobrist Hashing.

**Build:** recomputed keys, deterministic random tables, incremental updates, and debug cross-checks.

**Practice:** every special move, undo, and changes to rights or en passant state.

**Explain from memory:** index collisions versus full-key collisions, and why a hash is not a mathematical proof of equality.

**Finish when:** all transition and restoration tests agree with full recomputation.

### Week 17 — Transposition tables

**Read:** CPW Transposition Table.

**Build:** simple entries, depth checks, bounds, replacement, and mate-score normalization.

**Practice:** tiny tables, forced collisions, repeated searches, and cache-on/cache-off comparisons.

**Explain from memory:** why a history-dependent draw must not become an unrestricted exact cache result.

**Finish when:** M8 passes.

### Week 18 and beyond — Controlled improvement

Choose one track: ordering, evaluation, representation, or selective search. Write the hypothesis before modifying the engine. Use the experimental workflow in [search and evaluation](search-and-evaluation.md).

Possible four-week cycle:

1. Study and specify a single technique.
2. Implement it behind a switch and add failure-focused tests.
3. Run controlled benchmarks and paired matches.
4. Review evidence, keep or revert, and document what you learned.

## If you have less time

At three hours per week, spread each planned week across two or three actual weeks. Preserve the sequence: reading, implementation, verification, explanation. Avoid splitting attention between unfinished move generation and advanced search.

## If you get stuck

1. Reproduce the failure with one command.
2. Remove unrelated features from the experiment.
3. Write the smallest state transition involved.
4. Compare expected and actual intermediate state.
5. Ask for a test or hint using the [debugging template](debugging.md#bug-report-template).

Timebox unstructured investigation. After a focused period with no new evidence, change the experiment rather than repeatedly rereading the same code.

## Weekly review

Answer these questions before choosing the next task:

- What can the program do that it could not do last week?
- Which tests establish that behavior?
- Which assumption turned out to be wrong?
- Can I explain the implementation without opening the source?
- Which limitations are intentional?
- What is the smallest useful next change?

If you cannot explain a feature, schedule a review exercise before adding another layer on top of it.
