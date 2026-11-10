# Documentation guide

This is a curriculum for implementing your own C++ chess engine. It provides reasoning prompts, design tradeoffs, test expectations, and exit criteria rather than a ready-made engine.

## Intended outcome

The first complete version should implement standard chess, pass a substantial move-generation test suite, choose moves through a classical search, and communicate with an existing chess GUI through UCI. Its exact playing strength is not a promised milestone.

Chess960, graphical interfaces, neural-network training, parallel tree search, and opening databases are optional later projects. Keeping the initial scope small makes failures easier to explain.

## Reading routes

### Beginning the project

Read [roadmap](roadmap.md), [study plan](study-plan.md), and [C++ development](cpp-development.md). Then use [architecture](architecture.md) to write a short design note before creating position types.

### Implementing the rules

Keep [rules and edge cases](rules-and-edge-cases.md) and [testing](testing.md) open together. For every rule, create a small example that permits the move and another that forbids it.

### Implementing search

Read [search and evaluation](search-and-evaluation.md). Keep the tested rules layer stable while experimenting with search. Re-run the relevant regression suite whenever an experiment changes shared position state.

### Investigating a failure

Use [debugging](debugging.md). Reduce the problem to a short move sequence or small position before reading a large amount of code.

### Finding deeper explanations

Use [references](references.md). Read the material for your current milestone, try explaining it from memory, and implement your understanding. Exhaustively reading the entire Chess Programming Wiki before starting is unnecessary.

## How to interpret these documents

- **Deliverable:** something observable that you create.
- **Invariant:** a condition that must hold whenever the relevant state is valid.
- **Acceptance gate:** evidence required before moving forward.
- **Exercise:** a task for building understanding; it may be a written explanation rather than code.
- **Experiment:** a reversible change evaluated against a known baseline.
- **Reference oracle:** an independent implementation or published result used for comparison.

The schedules are planning estimates for someone who already knows basic programming. Progress is determined by the acceptance gates. If a gate takes longer than a week, move the schedule instead of weakening the gate.

## Proposed milestones

| Milestone | Observable result |
| --- | --- |
| M0 | A clean build, a working test runner, and useful debugger access |
| M1 | Positions load, display, and export correctly |
| M2 | Complete legal move generation with reversible state changes |
| M3 | Published perft tests and independent move-set comparisons pass |
| M4 | Terminal states and game-history policies are documented and tested |
| M5 | A shallow exhaustive engine produces explainable scores |
| M6 | Alpha-beta and quiescence pass their own correctness checks |
| M7 | The engine can play complete games through UCI |
| M8 | Incremental hashing and transposition-table use are verified |
| M9 | Strength or speed improvements are supported by experiments |

## Maintain your own evidence

Create implementation notes as you work. Suggested fields:

```text
Milestone:
Design decision:
Alternatives considered:
Invariant:
Tests added:
Failure discovered:
Cause:
Evidence that the fix works:
Remaining limitations:
```

Do not mark a feature complete simply because it returns plausible moves. The most useful documentation is the explanation of why you believe it is correct.

## First work session

1. Confirm that a C++20 compiler, CMake, and a debugger are available.
2. Write the smallest program and test executable yourself.
3. Deliberately make one test fail and read its output.
4. Stop inside the debugger and inspect a local variable.
5. Write your square-numbering convention on paper.

At the end of the session, you should be able to reproduce your setup from a clean build directory. No chess moves need to exist yet.
