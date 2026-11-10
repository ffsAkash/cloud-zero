# Cloud Zero

A chess engine learning project in C++20. The engine will be written by its author, from the rules of chess through search and evaluation. This repository begins with the study material and implementation plan.

## Current status

The engine is not implemented yet. There are no build targets, dependencies, or executable commands to run. The documentation describes proposed work and acceptance criteria; it does not imply those features already exist.

## First goal

Build a single-threaded engine for standard chess that generates legal moves, passes reference move-generation tests, searches positions, and plays through the Universal Chess Interface (UCI).

The proposed starting point is C++20, a 64-square array, classical evaluation, and an exhaustive search that gradually gains alpha-beta pruning and other improvements. Representation changes and advanced search techniques come after a tested baseline.

## Start here

1. Read the [documentation guide](docs/readme.md).
2. Work through the [roadmap](docs/roadmap.md) in order.
3. Follow the [study plan](docs/study-plan.md) at a sustainable pace.
4. Prepare your environment using the [C++ development guide](docs/cpp-development.md).
5. Write your initial design decisions using [architecture and invariants](docs/architecture.md).

Your first deliverable is a working build and test executable, followed by square conversion, a board display, and FEN parsing/export. Move generation comes afterward.

## Documentation map

| Document | Purpose |
| --- | --- |
| [Roadmap](docs/roadmap.md) | Implementation order, exercises, dependencies, and completion gates |
| [Study plan](docs/study-plan.md) | Weekly schedule, session structure, and progress assessment |
| [C++ development](docs/cpp-development.md) | Language prerequisites, build tools, diagnostics, and debugging |
| [Architecture](docs/architecture.md) | Position state, boundaries, score conventions, and invariants |
| [Rules and edge cases](docs/rules-and-edge-cases.md) | Legal moves, special moves, history, and draw semantics |
| [Testing](docs/testing.md) | Fixtures, perft counts, property tests, and differential testing |
| [Search and evaluation](docs/search-and-evaluation.md) | Search concepts, evaluator design, caching, and experimental improvements |
| [Debugging and collaboration](docs/debugging.md) | Reproducible bug reports, diagnostic workflows, and assistance boundaries |
| [References](docs/references.md) | Annotated reading list organized by implementation stage |

## How assistance works

You own the implementation. The default role of an assistant is to help with explanations, test design, code review, and bug isolation. Hints and counterexamples come before solutions. Engine code, patches, and scaffolding should be provided when you explicitly request them.

For debugging, include a FEN, relevant move history, expected and actual behavior, a reproducible command, and the smallest relevant code section. See the [bug report template](docs/debugging.md#bug-report-template).

## Development principles

- Correct rules before stronger search.
- Inspectable state before compact encodings.
- Reproducible tests before performance claims.
- One experimental change at a time.
- Preserve a simple reference implementation when introducing optimizations.
- Record every fixed bug as a regression test.

## License

See [LICENSE](LICENSE).
