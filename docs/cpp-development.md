# C++ development guide

The proposed language standard is C++20. This guide describes a development setup to implement yourself. The repository does not yet contain the build files or targets used in the examples.

## Language knowledge to acquire first

| Topic | Why the engine needs it | Exercise |
| --- | --- | --- |
| Enums and small structs | Express colors, pieces, moves, and scores clearly | Model a square and reject invalid coordinates |
| References and const correctness | Separate inspection from mutation | Identify which position operations should be const |
| Value semantics | Copy positions and compare child states | Explain which members are copied and whether any pointers alias |
| Arrays and vectors | Store boards, move lists, and history | Predict when a vector reference becomes invalid |
| Recursion | Traverse the move tree | Trace a tiny tree with depth and ply shown separately |
| Object lifetime and RAII | Restore state and own resources safely | Explain who owns a search worker and when it stops |
| Integer arithmetic | Encode squares and scores safely | Find an expression whose unsigned subtraction wraps |
| Bitwise operations | Represent flags and later bitboards | Set, clear, and test one flag without changing the others |
| Streams and strings | Parse FEN and UCI commands | Reject an incomplete line without corrupting a position |
| Chrono and atomics | Measure time and request a search stop | Distinguish a monotonic clock from wall-clock time |

[LearnCpp](https://www.learncpp.com/) is a useful lesson-by-lesson starting point. Learn advanced language features when the engine provides a concrete reason to use them.

## Suggested project organization

Once implementation starts, a possible organization is:

```text
src/          production code
include/      public or shared headers, if useful
tests/        test executable and test helpers
tests/data/   reference positions and expected results
tools/        comparison and benchmark utilities
docs/         this curriculum and your design notes
build/        generated local build directories, ignored by Git
```

This is a proposal, not an existing layout. A small project can keep headers next to their implementation files. Avoid creating empty abstractions merely to match a directory diagram.

## Build configurations

Create at least two configurations:

- **Debug:** symbols, assertions, modest or no optimization, and development diagnostics.
- **Release:** optimization for measurement and actual play.

A sanitizer configuration can be a separate build. For GCC or Clang, typical development flags include `-g`, `-fsanitize=address,undefined`, and `-fno-omit-frame-pointer`. Configure both compilation and linking appropriately. See the official [AddressSanitizer documentation](https://clang.llvm.org/docs/AddressSanitizer.html) for supported options and limitations.

After you write the CMake targets and register tests, a typical workflow might be:

```sh
cmake -S . -B build/debug -DCMAKE_BUILD_TYPE=Debug
cmake --build build/debug
ctest --test-dir build/debug --output-on-failure
```

These commands are examples for a future CMake setup. Multi-configuration generators need configuration selection at build/test time. Consult the [CMake tutorial](https://cmake.org/cmake/help/latest/guide/tutorial/index.html).

## Warnings and assertions

Start with `-Wall -Wextra -Wpedantic` when using GCC or Clang. Understand warnings and fix their causes. Add conversion and signedness warnings once the initial output is manageable.

Use assertions for programmer assumptions such as valid square ranges and coherent king locations. Use ordinary error handling for invalid external input. Assertions may disappear when `NDEBUG` is enabled; test failures must still be reported in configurations where tests are expected to run.

Assertions are especially useful after make/unmake while developing. Expensive full-state consistency checks can remain in debug builds and be disabled for performance measurements.

## Common C++ traps in this project

### Signed and unsigned arithmetic

A square stored in an unsigned type can wrap when moving toward smaller indices. Validate coordinates using an arithmetic type that can represent intermediate negatives, then convert after checking.

### Bit shifts

For future bitboards, use an unsigned 64-bit value for masks and validate shift counts. Shifting by a negative count or by at least the type width is invalid. A plain integer literal may have a narrower type than intended.

### Score negation

Negamax negates scores and bounds. Do not use the minimum signed integer as a value you intend to negate. Reserve safe headroom for infinity, mate scores, ply offsets, and evaluator output.

### Invalidated references

Growing a vector can invalidate references to its elements. A reference to an undo record, move, or history entry must not outlive a reallocation. Be cautious when retaining references across recursive calls that may grow shared storage.

### Struct comparison

Use field equality for positions and undo checks. Raw memory comparison can observe padding rather than logical differences. Hash equality is also insufficient as the only state-restoration check.

### Hidden state and globals

Mutable global boards, counters, and search results make isolated tests difficult. Keep ownership explicit. Immutable attack tables can be shared without making the current position global.

### Incomplete cleanup on return

Cutoffs, cancellation, terminal returns, and errors can all bypass a manually placed undo call. Structure control flow carefully. RAII may help, but first establish clear ownership and restoration rules.

### Data races

One search worker plus a responsive input loop can still introduce concurrency bugs. Use a properly synchronized stop signal. Give the search worker exclusive access to its mutable position. Do not resize a live transposition table while another thread probes it.

## Using a debugger deliberately

1. Reproduce the smallest failure with a fixed input.
2. Break immediately before the incorrect transition.
3. Inspect the position, side to move, move kind, and saved state.
4. Step through the mutation once.
5. Compare each changed field against your prediction.
6. Step through undo and compare again.

A conditional breakpoint on a specific FEN-derived state, square, or move can be more useful than stopping at every recursive call. Watchpoints can reveal who unexpectedly changes a castling flag or king square.

## Testing tools

Choose a maintained C++ test framework if convenient, or begin with an explicit runner that reports failed conditions and exits unsuccessfully. Test cases should print enough context to reproduce a failure: FEN, move, depth, seed, and expected/actual values where relevant.

Use unit tests for small operations, property tests for invariants across many states, and differential tests against independent implementations. Their roles are explained in [testing](testing.md).

## Measurement discipline

- Compare release builds with the same compiler settings.
- Record hardware, compiler, flags, thread count, hash size, and test input.
- Avoid logging inside the measured search loop.
- Distinguish elapsed time, CPU time, nodes, and nodes per second.
- Repeat short measurements enough to understand noise.
- Profile before rewriting a subsystem for speed.

The first performance goal is repeatability. A small apparent speedup can disappear when measurement conditions change.
