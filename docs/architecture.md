# Architecture and invariants

This document is a design worksheet. Write down your choices before implementation, then update the notes when evidence changes them. A small engine benefits from clear responsibilities more than from a large class hierarchy.

## Suggested boundaries

| Responsibility | Inputs and outputs | Avoid mixing in |
| --- | --- | --- |
| Position and notation | Board state, FEN, readable display | Search policy and GUI concerns |
| Attacks | Position, square, attacking color | Recursive legal-move generation |
| Move generation | Position to candidate/legal moves | Evaluation and time management |
| State transition | Position and move to child/undo state | Protocol parsing |
| Game history | Actual past positions and moves | Unrestricted cache assumptions |
| Evaluation | Position to bounded static score | Search control flow |
| Search | Position and limits to result/status | Direct parsing of raw input lines |
| UCI | Text commands and structured requests/results | Piece movement rules |
| Tests and tools | Reproducible inputs and comparisons | Production-only dependencies on test oracles |

Functions and plain structs can express these boundaries. Separate classes are optional.

## Position state

The initial 64-square representation should make occupied and empty squares explicit. Document whether an empty piece has a meaningful color and prevent accidental use of that color.

Store or derive:

- Piece placement.
- Side to move.
- Four standard-chess castling rights.
- En passant target or absence.
- Halfmove clock.
- Fullmove number.
- King squares, if cached.

History is additional state. A standalone position can be loaded from FEN without knowing previous occurrences. Search must understand this limitation rather than inventing history.

### Position invariants

For accepted ordinary legal positions:

- Exactly one king exists for each color.
- Every square contains one recognized piece or is empty.
- Cached king squares match board placement.
- Side to move is a valid color.
- No pawn remains on a promotion rank after a completed move.
- Counters are within your chosen representable range.
- Redundant representations agree, if you later add bitboards or piece lists.

Some parser tests intentionally use invalid positions. Decide whether to reject them or support a separate relaxed mode. Do not silently let test conveniences weaken production invariants.

## Square mapping

A suggested convention is `a1 = 0`, `b1 = 1`, and `h8 = 63`. File and rank then follow directly from the index. The printed board normally starts with rank eight, so display order differs from increasing index order.

Write tests for all squares. File/rank conversion helpers should be small and easy to inspect. Their correctness affects every later subsystem.

## Move representation

Begin with named fields rather than a packed integer. A move needs enough information to distinguish:

- Ordinary moves and captures.
- Double pawn pushes, if treated specially.
- En passant.
- Castling.
- Promotion piece type.

Decide which facts are encoded and which are read from the current position. Avoid redundant fields that can disagree unless you assert their consistency.

UCI coordinate notation usually identifies a move with source, destination, and optional promotion, such as `e2e4` or `a7a8n`. Resolve parsed text against generated legal moves. This lets the rules layer determine special move kinds.

## Transition contract

Define whether make-move accepts legal moves only or also pseudo-legal candidates. Both designs can work, but callers and tests must know which guarantees apply.

One useful early contract is:

1. Candidate generation obeys movement and occupancy rules.
2. Applying a candidate updates all state consistently.
3. Legality checking rejects candidates leaving the mover's king attacked.
4. Undo restores the original state even for rejected candidates.

Castling's transit conditions still need explicit checks. Keep a saved value of the moving color when side to move changes.

### Undo information

Consider saving:

- Captured piece and its actual square.
- Previous castling rights.
- Previous en passant target.
- Previous counters.
- Previous cached king locations if necessary.
- Previous hash or enough information to restore it.
- Previous incremental evaluation fields when introduced.

The move itself supplies some reversible information. The undo record supplies information lost during the forward transition.

The central property is:

> Undoing a completed make operation restores every semantically meaningful field, including after a search cutoff or cancellation.

## Copying and in-place mutation

Copied child positions offer a simple early implementation and a useful reference. In-place make/unmake reduces copying but increases the importance of control-flow correctness.

During migration, compare both child states for the same parent and move. Keep the reference path in tests if its maintenance cost remains low.

If a position owns history, be explicit about copy cost and ownership. Copying a position should not accidentally share mutable history through pointers.

## Score contract

Choose a single search score convention, such as positive meaning favorable to the side to move. Document whether a separate static evaluator produces White-relative scores before conversion.

Partition the score range conceptually into:

- Ordinary evaluation scores.
- Mate scores with root-relative distance.
- Search bounds beyond valid returned scores.

Keep enough headroom for safe negation and ply adjustment. An aborted search requires a separate status; a numerical sentinel must not accidentally become a valid score or transposition-table entry.

## Search result

A useful result concept includes:

- Completion or interruption status.
- Best legal root move, when one exists.
- Score with an explicitly defined interpretation.
- Completed depth.
- Node counts and elapsed time.
- Principal variation, if available and validated.

An interrupted iteration must not replace a completed result with incomplete data. A root with no legal moves must be representable without inventing a move.

## Hash identity and history

Position hashing usually covers pieces, side to move, castling rights, and an en passant convention. Repetition equality and transposition-table reuse are related but distinct concerns.

For repetition, irrelevant en passant metadata must not create a false distinction. For search caching, halfmove counters and repetition history can affect results even when an ordinary board key matches.

Choose a policy before implementing the table. Options include conservative restrictions on draw-sensitive cutoffs, additional context, or a more explicit key design. Test the chosen policy; simply adding a table lookup is insufficient.

## UCI ownership and responsiveness

The protocol layer translates text into structured requests. Search should receive a position snapshot and search limits rather than raw command strings.

If you use an input thread and search worker:

- The worker owns its mutable search position.
- A synchronized signal requests cancellation.
- Search results cross the boundary through defined synchronization.
- New position or hash-size commands do not mutate live search state unexpectedly.
- Shutdown joins the worker before destroying shared resources.

This is control concurrency, not parallel tree search. Keep the latter as a separate future project.

## Design-note template

```text
Decision:
Problem it solves:
Alternatives:
Chosen representation/contract:
Invariants:
Failure modes:
Tests:
Known limitations:
What evidence would make me revisit it:
```

Complete one note for square mapping, move encoding, transition ownership, score perspective, and history handling. Short notes that expose assumptions are more useful than elaborate diagrams without contracts.
