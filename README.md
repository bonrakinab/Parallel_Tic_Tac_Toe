# Parallel Tic-Tac-Toe with OpenMP

An academic C project exploring **serial vs parallel game-tree search** through Tic-Tac-Toe. The repository contains multiple implementations of the same game/search problem, including an OpenMP version that parallelizes evaluation of candidate moves at the root of the search tree.

## Project goal

Tic-Tac-Toe has a small enough state space to make it useful for experimenting with exhaustive search and parallel computing. This project uses the game to compare conventional execution with a parallelized implementation and to expose runtime information such as thread IDs and execution time.

## Repository contents

```text
Parallel_Tic_Tac_Toe/
├── parallelTic.c   # OpenMP-based parallel search
├── serialTic.c     # serial implementation
├── tic.c           # regular/base implementation
├── .gitignore
└── README.md
```

## How the search works

The program represents the board as a 3×3 matrix and recursively evaluates possible moves.

Core components include:

- Board creation and cloning
- Move generation for empty cells
- Win/draw detection across rows, columns, and diagonals
- Recursive score evaluation
- Alpha/beta bounds to avoid unnecessary search
- Automatic move selection from evaluated game states

At the top level of the parallel version, candidate moves are distributed using an OpenMP parallel loop:

```c
#pragma omp parallel for ... schedule(guided, CHUNK_SIZE)
```

Each worker evaluates a cloned board for a candidate move. A critical section protects updates to the best score and selected move.

## Parallel execution

The OpenMP implementation reports runtime information such as:

- OpenMP thread limit
- Whether execution is currently inside a parallel region
- Active thread IDs
- OpenMP timer resolution
- Execution time

The root-level search is parallelized while deeper recursive search continues serially. This keeps the experiment straightforward while demonstrating how independent branches of a search tree can be evaluated concurrently.

## Build and run

A compiler with OpenMP support is required for `parallelTic.c`.

### GCC

```bash
gcc parallelTic.c -fopenmp -O2 -o parallel-tic
./parallel-tic
```

Serial version:

```bash
gcc serialTic.c -O2 -o serial-tic
./serial-tic
```

Base implementation:

```bash
gcc tic.c -O2 -o tic
./tic
```

You can control the number of OpenMP threads through the environment, for example:

```bash
OMP_NUM_THREADS=4 ./parallel-tic
```

## Key concepts demonstrated

| Area | Implementation |
|---|---|
| Game state | 3×3 board structure |
| Search | Recursive game-tree evaluation |
| Optimization | Alpha/beta bounds |
| Parallelism | OpenMP `parallel for` |
| Scheduling | Guided scheduling with a chunk size |
| Synchronization | OpenMP critical section / barriers |
| Benchmarking | CPU execution-time reporting |

## Why parallelize game-tree search?

Moves available from the same game state are largely independent until their scores need to be compared. That makes top-level branches natural candidates for concurrent evaluation:

```text
Current board
    │
    ├── Move A ── evaluate subtree ──┐
    ├── Move B ── evaluate subtree ──┼── choose best score
    └── Move C ── evaluate subtree ──┘
```

For a tiny game such as Tic-Tac-Toe, OpenMP overhead can be significant relative to the workload. The project is therefore best understood as a **parallel-programming demonstration**, not as a claim that Tic-Tac-Toe needs parallel computing.

## Possible improvements

- Add repeatable benchmarks across several thread counts
- Report wall-clock time with `omp_get_wtime()`
- Compare serial and parallel speedup in a table
- Add deterministic test boards
- Clean up synchronization directives and memory management
- Generalize the board dimensions
- Extend the experiment to a larger search problem where parallelism provides clearer speedup

## Tech stack

- **Language:** C
- **Parallel API:** OpenMP
- **Topics:** parallel computing, recursive search, alpha-beta pruning, synchronization, benchmarking

---

This repository is preserved as a parallel-computing experiment demonstrating how independent branches of a game tree can be evaluated concurrently with OpenMP.
