# Annotated references and reading order

These references were checked while preparing the project curriculum. URLs and project interfaces can change; consult each project's own documentation before installing tools or copying commands.

The implementation order below is a suggested learning sequence. It is not a claim that there is one mandatory engine architecture.

## How to read technical material

For each topic:

1. Read the concept and its assumptions.
2. Work a tiny example on paper.
3. Close the reference and explain the idea from memory.
4. Design tests before translating the idea into C++.
5. Implement your understanding.
6. Return to the reference when an experiment exposes a gap.

Reading another engine can be useful after your own attempt. If your goal is independent implementation, avoid line-by-line transcription of a complete engine tutorial.

## Stage A — C++ and development tools

### LearnCpp

[LearnCpp](https://www.learncpp.com/)

Use this to close language gaps: initialization, enums, structs, references, object lifetime, containers, const correctness, and debugging. It is a lesson-oriented introduction to modern C++.

**Read first:** compilation and linking, functions, arrays, structs, references, and lifetime. **Read later:** templates, advanced ownership patterns, concurrency, and optimization topics when needed.

**Learning question:** Can you explain exactly how a position copy owns its data?

### CMake's official tutorial

[CMake tutorial](https://cmake.org/cmake/help/latest/guide/tutorial/index.html)

Use it to create targets, specify language requirements, and learn the build workflow. Read enough to support your actual project rather than treating build-system expertise as a prerequisite for chess programming.

**Learning question:** Can a clean build directory reproduce your executable and test runner?

### Clang's AddressSanitizer documentation

[AddressSanitizer](https://clang.llvm.org/docs/AddressSanitizer.html)

Use the official documentation to understand instrumentation, flags, diagnostics, and limitations. Pair it with undefined-behavior diagnostics in your development configuration where supported.

**Learning question:** Can you distinguish a symptom location from the earlier operation that corrupted state?

## Stage B — Chess rules and position state

### FIDE Laws of Chess

[FIDE Laws of Chess taking effect from 1 January 2023](https://handbook.fide.com/chapter/E012023)

Read article 3 for movement, attacks, and special moves; article 5 for game completion; and article 9 for draws. This is the rules source linked by the curriculum. Recheck the handbook before claiming compliance with a different or newly issued rules edition.

**Learning question:** Which decisions require only a position, and which require game history or a claim procedure?

### Chess Programming Wiki: Getting Started

[Getting Started](https://chessprogramming.org/Getting_Started)

Read this once for orientation. It emphasizes a correct rules foundation and testing. Then study focused pages as you implement each subsystem.

**Learning question:** What evidence should you require before adding search?

### Board representation

[Board Representation](https://chessprogramming.org/Board_Representation)

Compare square-centric and piece-centric approaches. The curriculum proposes a 64-square array initially because it is easy to inspect. Bitboards remain a later option.

**Learning question:** Which operations are easy in your representation, and where does it require extra work?

### FEN

[Forsyth-Edwards Notation](https://chessprogramming.org/Forsyth-Edwards_Notation)

Use this for position import/export and reproducible bug reports. Pay attention to metadata and en passant conventions.

**Learning question:** Which state is absent from FEN even though a game adjudicator may need it?

## Stage C — Moves and verification

### Move generation

[Move Generation](https://chessprogramming.org/Move_Generation)

Study candidate generation, legality, and attack relationships. Begin with a simple approach whose intermediate state can be printed and compared.

**Learning question:** Can your attack query operate without calling your legal move generator?

### State transitions

[Make Move](https://chessprogramming.org/Make_Move) and [Unmake Move](https://chessprogramming.org/Unmake_Move)

Use these to understand which changes are reversible and which require saved information. Relate every field to a concrete transition test.

**Learning question:** What information is destroyed by a capture or castling-right update?

### Perft reference positions

[Perft Results](https://chessprogramming.org/Perft_Results)

Use the published positions and exact counts to verify move trees. The repository's [testing guide](testing.md) reproduces a practical starter subset.

**Learning question:** How do you reduce a count mismatch to one illegal or missing move?

### Independent C++ rules implementation

[Disservin's chess-library](https://github.com/Disservin/chess-library)

Use as a separate test oracle for legal moves and resulting state. Keep its rules implementation outside the production engine if your goal is to implement the rules yourself. Record the oracle version for reproducibility.

**Learning question:** How do you normalize move notation and FEN metadata before comparison?

## Stage D — Evaluation and search

### Evaluation

[Evaluation](https://chessprogramming.org/Evaluation)

Start with material, then add positional terms gradually. A decomposition of evaluation terms is often more educational than a single final score.

**Learning question:** Which carefully chosen position isolates the effect of one term?

### Negamax

[Negamax](https://chessprogramming.org/Negamax)

Study score symmetry and then solve small trees by hand. Verify the sign convention at every level before searching chess positions deeply.

**Learning question:** Whose advantage does each returned score describe?

### Alpha-beta

[Alpha-Beta](https://chessprogramming.org/Alpha-Beta)

Study bounds and cutoff conditions. Preserve an exhaustive implementation to compare exact full-window results.

**Learning question:** When does a return value establish only an upper or lower bound?

### Quiescence

[Quiescence Search](https://chessprogramming.org/Quiescence_Search)

Study tactical horizon effects, stand pat, ordering, and check handling. Keep the first implementation conservative.

**Learning question:** What should happen when the only legal check evasion is quiet?

### Iterative deepening

[Iterative Deepening](https://chessprogramming.org/Iterative_Deepening)

Study why repeated shallower searches can improve ordering and support interruption.

**Learning question:** Which result remains trustworthy when the current iteration is interrupted?

## Stage E — Protocol, caching, and measurement

### UCI specification

[Shredder's UCI page](https://www.shredderchess.com/chess-features/uci-universal-chess-interface.html)

The page links the protocol specification download. Use that specification for command syntax, lifecycle, and output rules. The web page alone is not the full protocol document.

**Learning question:** How does your process continue handling commands while a search is running?

### Zobrist hashing

[Zobrist Hashing](https://chessprogramming.org/Zobrist_Hashing)

Study position keys and incremental XOR updates. Implement independent full recomputation before trusting updates.

**Learning question:** Which state fields affect your key, and which result-affecting fields are omitted?

### Transposition tables

[Transposition Table](https://chessprogramming.org/Transposition_Table)

Study depth requirements, exact/upper/lower bounds, collisions, and replacement. Add history-sensitive results and mate normalization to your own design review.

**Learning question:** Why is a matching key insufficient to justify every cached cutoff?

### Fastchess

[Fastchess](https://github.com/Disservin/fastchess)

A command-line tool for engine matches, with a UCI compliance checker described in its repository documentation. Use it after the process lifecycle works reliably.

**Learning question:** What do your match conditions control, and how large is the uncertainty in the result?

## Stage F — Later study

### Bitboards

[Bitboards](https://chessprogramming.org/Bitboards)

Study set operations, masks, and attack generation after you have a trusted reference path. A representation migration should preserve exact legal behavior.

**Learning question:** How will you detect a mismatch between board arrays and bitboards if both are maintained?

### Official Stockfish source

[Stockfish](https://github.com/official-stockfish/Stockfish)

Read a narrowly selected subsystem after implementing your own version. A mature engine contains interacting optimizations and conventions that can be hard to understand in isolation. Treat source study as an opportunity to explain design choices, not a requirement to reproduce its architecture.

**Learning question:** What assumptions does a routine rely on that are established elsewhere in the program?

## Optional research reading path

Once M8 is stable, follow the original papers and references linked from the CPW pages for topics such as alpha-beta, principal variation search, static exchange evaluation, pruning, and statistical engine testing.

For a paper, record:

```text
Problem:
Baseline method:
New idea:
Assumptions:
Experimental setup:
Reported evidence:
What I would need to reproduce it:
How it might interact with my engine:
```

You do not need a large bibliography before implementing the first move generator. A small number of well-understood references and reproducible experiments will carry the project further.
