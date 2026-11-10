# Debugging and collaboration

The author owns the implementation. Assistance should strengthen your ability to reason about the engine and find the next bug yourself.

## Default assistance boundaries

Unless you ask for something else, useful assistance includes:

- Explaining a rule, invariant, or algorithm.
- Designing tests and predicting expected outcomes.
- Reviewing a design or a small code section.
- Helping reduce a failure to a minimal reproducer.
- Suggesting instrumentation and experiments.
- Giving progressively more specific hints.

Implementation code, scaffolding, automated patches, and broad rewrites should be requested explicitly. Creating these learning documents does not request engine implementation.

When asking for review, specify the level of help you want:

```text
Hints only; do not identify the exact line yet.
Review the reasoning and suggest tests.
Identify the defect, but let me write the fix.
Review my fix and regression test.
Show a minimal implementation example.
```

## Debugging cycle

1. **Observe:** state exactly what differs from expectation.
2. **Reproduce:** save an input and command that produce the same failure.
3. **Reduce:** remove moves, pieces, options, and depth that are unnecessary.
4. **Predict:** write what each relevant state field should be.
5. **Measure:** inspect actual intermediate state.
6. **Hypothesize:** name one cause that explains the evidence.
7. **Discriminate:** run a test that separates that cause from alternatives.
8. **Fix:** change the smallest responsible behavior.
9. **Preserve:** add a regression case and run related tests.

Avoid changing several unrelated functions before testing. A disappearing symptom is not evidence that every change was needed.

## Bug report template

```text
Title:
Milestone/feature:
Engine version or commit:
Compiler/build mode:

Starting FEN:
Actual prior move history, if relevant:
Reproduction command/input:
Depth, node limit, or time limit:
Random seed and generated move list, if relevant:

Expected behavior:
Source or reasoning for the expectation:
Actual behavior:
Smallest known failing case:

Feature switches:
Relevant code section:
Diagnostics or sanitizer output:
Experiments already tried and their results:
Current hypothesis:

Help wanted: tests / hints / explanation / defect identification / patch
```

For perft, attach divide output and the first mismatching depth. For search, attach root-move scores and indicate whether quiescence, caching, and selective pruning are enabled.

## Diagnose by failure class

### Board orientation or square conversion

**Symptoms:** pawn movement goes the wrong direction, FEN display is vertically flipped, or edge moves wrap.

**Next experiment:** print every square index alongside its coordinate. Test all conversions. Inspect a single pawn for each color before a complete initial position.

**Likely assumptions:** display order versus storage order, signed offsets, rank numbering, and file bounds.

### Unexpected legal moves

**Symptoms:** king walks into check, a pinned piece moves illegally, or a legal block is missing.

**Next experiment:** print candidate moves separately from legality-filtered moves. For one candidate, inspect the complete child position and attack result for the moving side's king.

**Likely assumptions:** wrong color after side changes, attack detection depending on legal moves, stale king squares, or incorrect capture occupancy.

### Special-move errors

**Symptoms:** perft fails only on positions with castling, promotions, or en passant.

**Next experiment:** isolate one special move and compare every changed field against a written checklist. Undo it immediately and compare the full parent.

**Likely assumptions:** wrong captured square, omitted promotion type, lost rights not restored, missing transit-square checks, or rights inferred from piece placement.

### Perft mismatch

**Symptoms:** a published total differs.

**Next experiment:** locate the shallowest failing depth, compare move sets, then follow one differing divide branch. Inspect the child FEN and state before recursing further.

**Likely assumptions:** a missing/duplicate move or incorrect state transition. Avoid investigating search evaluation: standard perft should not use it.

### Failure depends on move order

**Symptoms:** perft or exhaustive-search scores change when move order changes.

**Next experiment:** snapshot the parent before and after each child. Disable caching and verify that each sibling begins with identical state.

**Likely assumptions:** incomplete undo, reused move buffers, invalidated references, or uninitialized state. For advanced selective search, order dependence may have additional algorithmic causes, so first reproduce in the simple reference path.

### Engine prefers losing material

**Symptoms:** obvious losing captures or nonsensical Black-to-move play.

**Next experiment:** disable positional terms, quiescence, and caching as needed to isolate the layer. Print material scores and root move scores. Trace one line by hand.

**Likely assumptions:** score perspective, negation, wrong window transformation, zero-initialized best score, or terminal handling after the horizon check.

### Quiescence reports mate incorrectly

**Symptoms:** an in-check position with a quiet escape is labelled mate.

**Next experiment:** compare the quiescence move list to the full legal evasion list using fixture R6 from [testing](testing.md).

**Likely assumptions:** captures-only generation while in check or stand pat accepted before check handling.

### Hash-dependent behavior

**Symptoms:** failures appear only with caching enabled or with a particular hash size.

**Next experiment:** compare incremental and recomputed keys at every transition. Use a tiny table. Log entry key, depth, bound, score, and retrieved move for the first divergent node.

**Likely assumptions:** index collisions mistaken for key matches, incorrect bounds, stale scores, mate normalization, or history-sensitive draw reuse.

### Crashes or release-only failures

**Symptoms:** optimized builds fail while debug builds seem fine, or crashes move when logging changes.

**Next experiment:** run the same reproducer with sanitizers and warnings enabled. Inspect lifetimes, bounds, recursion depth, and signed arithmetic.

**Likely assumptions:** undefined behavior, invalidated references, dangling pointers, or stack exhaustion. Do not regard a lower optimization setting as a completed fix.

### Stop or shutdown hangs

**Symptoms:** the process ignores stop, emits a stale move, or hangs on quit.

**Next experiment:** record the protocol transcript and worker lifecycle. Test stop before depth one completes and quit during search. Identify which thread can access each object.

**Likely assumptions:** blocked input loop, unsynchronized cancellation, worker join ordering, or a search result published for an obsolete position.

## Instrumentation worth building

Add debug-only facilities as needed:

- Board and complete metadata dump.
- Named move-kind output.
- Candidate and legal move lists.
- State consistency assertions.
- Make/unmake snapshots.
- Full hash recomputation checks.
- Root score tables.
- Search trace restricted to a chosen move prefix.
- Transposition-table probe/store logs for a chosen key.
- Protocol input/output transcript.

Keep logging off the UCI output stream unless it conforms to the protocol. Restrict trace output to the smallest relevant branch; millions of undifferentiated lines rarely help.

## Review your own fix

Before requesting another review, answer:

1. What invariant was violated?
2. Why did the old test suite miss it?
3. Does the new test fail on the old implementation?
4. Why does the change address the cause?
5. Which neighboring cases could it break?
6. Are there any new limitations?

The most valuable outcome is an explanation you can reuse on the next failure.
