# Search and evaluation study guide

Search asks which move to choose given a model of future play. Evaluation estimates positions where the search stops. Keep these responsibilities separate so you can test each one.

This guide provides contracts and exercises, not an engine implementation. Complete the legal-move and state-restoration milestones before relying on search results.

## 1. Define score meaning

A convenient negamax convention is positive for the side to move. Another valid design uses a White-relative evaluator and converts at the search boundary. Choose deliberately and test the conversion.

Use integer scores with a documented scale, often approximately centipawns for ordinary evaluations. Piece values are model parameters rather than immutable rules of chess.

Reserve distinct ranges for ordinary scores, mate-distance scores, and infinity bounds. Keep every negation and ply adjustment inside the representable range. Avoid using an ordinary numeric score to represent cancellation.

### Exercises

- Predict the material score for both sides to move without changing the pieces.
- Trace a score through three negations and describe its meaning at every level.
- Show why an initial best score of zero fails when every legal move loses.

## 2. Start with material evaluation

Begin with a deliberately small evaluator. Terminal mate and draw handling belong to search/game logic rather than ordinary material scoring.

For debugging, expose an evaluation breakdown showing each term before combination. Test adding/removing a piece, changing side to move, and color-swapping the position with the appropriate board transformation.

Read [CPW Evaluation](https://chessprogramming.org/Evaluation). Once material is correct, add piece-square terms and eventually separate middlegame/endgame values. Blend them using a documented phase measure and clamp phase if promotion can increase material beyond the original setup.

Potential later terms include pawn structure, passed pawns, mobility, bishop pair, rook activity, and king safety. Add one term at a time with positions where it should increase, decrease, and remain neutral.

Do not tune dozens of weights while basic sign conventions remain uncertain.

## 3. Learn exhaustive negamax on toy trees

Negamax uses zero-sum score symmetry to express both players' choices with the same recurrence. Read [CPW Negamax](https://chessprogramming.org/Negamax).

Before chess, construct small trees with known leaf scores. Include:

- One child.
- Several equally good children.
- All negative choices.
- A tempting immediate reward with a losing reply.
- A tree where the preferred child changes with depth.

Draw each tree and solve it by hand. The algorithm should agree without requiring any chess knowledge.

### Depth and ply

Depth is the remaining search horizon. Ply is the number of half-moves from the root. Extensions and reductions eventually make them diverge further. Mate-distance calculations usually need root-relative ply, not just remaining depth.

### Terminal handling

Recognize no-legal-move states before ordinary static evaluation, even at depth zero. Checkmate has a mate score; stalemate is a draw. Draw policies and history must be consistent with the rules layer.

### Acceptance evidence

Toy trees agree exactly. Simple chess fixtures work for both colors. Root state is unchanged afterward. You can explain why the selected move wins under the searched horizon.

## 4. Add alpha-beta without changing the game tree

Alpha-beta skips work when bounds establish that a branch cannot affect the current decision. Initially retain identical depth limits, evaluation, and terminal behavior to your exhaustive reference. Read [CPW Alpha-Beta](https://chessprogramming.org/Alpha-Beta).

Adopt one window convention, commonly a half-open interval, and distinguish exact results from bounds. In negamax the child window reverses and negates the parent's limits.

Choose fail-hard or fail-soft behavior and document it. Compare full-window root scores to exhaustive results. For narrow windows, test the promised bound rather than assuming every returned number is exact.

Move ordering affects work performed. Without selective techniques, it should not change the exact root value under the same complete search conditions. Equal-valued moves may be chosen differently.

### Exercises

- Mark the first legal cutoff in a hand-drawn tree.
- Reorder the children and count how much work changes.
- Explain why a cutoff value can be useful without being the exact position value.

## 5. Add quiescence carefully

A static horizon can stop in the middle of an exchange and reward a poisoned capture. Quiescence extends selected tactical continuations. Read [CPW Quiescence Search](https://chessprogramming.org/Quiescence_Search).

An initial design searches captures and promotions outside check. In check, search all legal evasions and forbid stand pat. Detect stalemate even when the tactical move list is empty. Use simple capture ordering and postpone aggressive quiescence pruning.

Stand pat is a search assumption, not a legal pass move. It is particularly delicate in zugzwang. Track quiescence nodes separately to expose unexpectedly large expansions.

Set defensive stack/ply limits, but define their behavior explicitly: a guard must not masquerade as a fully searched exact score, particularly in check. Repetition/cycle checks and cancellation still apply.

### Required exercises

- A capture loses to an immediate recapture.
- A promotion has no capture but matters tactically.
- The only check escape is quiet.
- The horizon is checkmate.
- The horizon is stalemate.

After quiescence changes the leaf procedure, earlier score comparisons must use the same procedure on both sides.

## 6. Add iterative deepening and principal variations

Search depth one, then two, then three, retaining completed results. Previous iterations can improve move ordering and provide a usable answer when time expires. Read [CPW Iterative Deepening](https://chessprogramming.org/Iterative_Deepening).

Establish a legal fallback immediately. Keep the current iteration's working result separate from the last completed result. Avoid publishing a partial iteration as an exact completed answer.

A principal variation is a sequence of expected best replies. Replay it from the root and verify every move is legal. Table-derived lines can become stale or form cycles; impose appropriate length limits and validate them.

### Cancellation contract

- Cancellation is separate from an ordinary score.
- Every made move is undone during unwinding.
- Incomplete nodes do not become unrestricted exact cache entries.
- The root result remains legal and tied to the correct position.

## 7. Add time management

Start with a fixed duration before allocating full-game clocks. Use a monotonic clock for elapsed search time.

Distinguish a soft deadline, where another iteration should generally not begin, from a hard deadline that requests interruption. Reserve some margin for communication and scheduling. Check time often enough to meet your measured responsiveness target without measuring it at every node unnecessarily.

Test very small budgets, stop before depth one finishes, no-legal-move roots, and quitting during search. Full-game clock allocation must account for time remaining, increment, and moves-to-go information when supplied.

One search worker can coexist with an input-handling thread. Keep ownership and synchronization explicit; this is separate from parallelizing the tree.

## 8. Introduce hashing and transposition tables

First implement full Zobrist recomputation, then verify incremental updates against it. Use deterministic tables so failures reproduce. Read [Zobrist Hashing](https://chessprogramming.org/Zobrist_Hashing).

Read [Transposition Table](https://chessprogramming.org/Transposition_Table) before adding cached cutoffs. Entries need a key check, search depth, bound type, score, and useful move. Implement a simple replacement policy before complicated aging or buckets.

Normalize mate scores so storage and retrieval at different plies preserve distance meaning. A stored lower bound cannot be treated as an exact score. A table-index collision must not impersonate the same position.

Validate cached moves before applying them. A shallow entry may help ordering while being insufficient for a score cutoff. History-dependent draws and move-count thresholds require an explicit reuse policy.

### Experiments

- Force collisions with a tiny table.
- Search controlled positions with caching enabled and disabled.
- Probe the same mate score at different root-relative plies.
- Compare an identical board reached with different histories and halfmove clocks.

## 9. Improve ordering before aggressive selectivity

Possible ordering sources include the previous principal variation, transposition-table move, captures, promotions, killer moves, and history scores.

Simple capture ordering may use the captured piece's value and attacker's value. It is a heuristic, not a guarantee that the exchange is favorable. Static exchange evaluation is a later refinement.

Evaluate ordering with nodes required for the same complete depth and leaf semantics. Nodes per second may fall while total search time improves because fewer nodes are needed.

## 10. Study advanced techniques as separate experiments

| Technique | Intended benefit | Important risk or test |
| --- | --- | --- |
| Principal variation search | Use narrow windows after a likely best move | Re-search conditions and bound correctness |
| Aspiration windows | Start near the previous iteration's score | Fail-high/fail-low recovery and interrupted re-search |
| Static exchange evaluation | Rank or filter tactical moves | X-rays, promotions, pins, and king captures |
| Null-move pruning | Detect positions likely to exceed beta | Zugzwang and state restoration |
| Futility pruning | Skip unlikely improvements | Quiet tactical moves and checks |
| Late-move reductions | Search less-promising moves more cheaply | Re-search after reduced searches improve the bound |
| Check extensions | Spend depth on forcing sequences | Search explosion and repeated checks |
| Bitboards | Speed attack and move calculations | Redundant-state consistency and edge masks |
| Parallel tree search | Use more CPU cores | Races, nondeterminism, and redundant work |
| Neural evaluation | Learn position features from data | Training, feature updates, reproducibility, and data quality |

PVS and aspiration windows aim to preserve complete-search results when their required re-searches finish. Selective pruning and reductions make additional approximations. Do not assess all these techniques with the same correctness claim.

## 11. Use an experiment record

```text
Hypothesis:
Baseline commit/version:
Candidate commit/version:
Feature switch:
Correctness checks:
Compiler and build flags:
Hardware and thread count:
Hash size:
Position/opening set and seed:
Time control or depth:
Nodes and time:
Tactical results:
Paired-match results and uncertainty:
Decision:
```

Use identical conditions and opening pairs with colors reversed. Preserve losses for analysis. Ten attractive wins are not sufficient evidence for a small strength gain.

[Fastchess](https://github.com/Disservin/fastchess) can run engine matches. Study statistical testing once you have a stable baseline and a repeatable experiment harness. The relevant question is whether the measured effect is distinguishable from noise under your chosen test conditions.
