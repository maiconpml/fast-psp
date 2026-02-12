# Just in Time Job Shop with Tabu Search

## Introduction

The **Just-In-Time (JIT) Job Shop Scheduling Problem** is a variation of the classical Job Shop Problem. In this variant, the goal is to schedule a set of jobs across multiple machines such that each operation is completed as close as possible to its specific due date. While traditional scheduling often focuses on minimizing the *makespan* (total completion time), JIT scheduling aims to minimize both **earliness** and **tardiness** penalties. This approach aligns with lean manufacturing principles, where both holding inventory (earliness) and missing deadlines (tardiness) are considered costly inefficiencies.

**Tabu Search** is a metaheuristic local search algorithm used for mathematical optimization. It enhances the performance of local search by using memory structures (the "Tabu List") to prevent the search from returning to recently visited solutions. This mechanism allows the algorithm to escape local optima by accepting non-improving moves, enabling a broader exploration of the solution space.

## Dependencies

This project requires the following libraries:

*   **IBM ILOG CPLEX Optimization Studio**: Used for optimal scheduling of operations and within specific neighborhood moves (CPLEX Relax).
*   **Boost C++ Libraries**: Specifically `program_options`, `timer`, `system`, and `chrono`.

> [!IMPORTANT]
> This project has been developed and tested exclusively on **Linux**. Compatibility with Windows or other operating systems is not guaranteed and may require significant adjustments to the build process and dependency management.

## Project Structure

*   `apps/`: Contains the entry point of the application (`main.cpp`).
*   `src/`: Contains the core logic (headers and implementation).
*   `instances/`: Example problem instances.
*   `cmake/`: CMake modules for finding dependencies.

## Compilation

### Debug Build
```bash
mkdir Debug && cd Debug
cmake -DCMAKE_BUILD_TYPE=Debug ..
make
```

### Release Build
```bash
mkdir Release && cd Release
cmake -DCMAKE_BUILD_TYPE=Release ..
make
```

## Running

```bash
./main [INSTANCE_PATH] [LOG_NAME] [Program Options]
```

### Program Options
*   `--help`: Show help.
*   `--instPath`: Path to the instance file.
*   `--name`: Name for the log.
*   `--maxSecs`: Maximum time in seconds (default: 300).
*   `--seed`: Random number generator seed (default: 13).
*   `--tenure`: Tabu tenure (default: 8).
*   `--initialjumpLimit`: Initial maximum iterations without improvement before backjump (default: 2500).
*   `--decreaseDivisor`: Decrease in `jumpLimit` each time a backjump occurs without improvement (default: 7).
*   `--bjSize`: Backjump list size (default: 20).
*   `--maxD`: Cycle size to stop search and force backjump (default: 100).
*   `--maxC`: Number of cycle repeats to force backjump (default: 2).
*   `--timeLog`: Show time of each new best solution found (default: true).
*   `--scaleTime`: Scaling factor for time, for testing (default: 1.0).
*   `--onlyMakesLowerBound`: Get lower bound values only (default: false).
*   `--schedulerType`: Type of scheduler used:
    *   1: Early as possible (fastest).
    *   2: Delaying when possible (better schedule but slower).
    *   3: Hybrid (1 when tardiness penalties dominate, 2 when earliness penalties dominate).
    *   4: CPLEX (slowest but optimal).
    (default: 1)
*   `--useSwapAllNIter`: Number of iterations without improvement to trigger the "Swap All" neighborhood (default: 1).
*   `--useCplexRelaxRatio`: Ratio between earliness and tardiness penalties to trigger the "CPLEX Relax" neighborhood (default: 0.25).

## Tabu Search

The current Tabu Search implementation uses three neighborhoods:

1.  **Swap All**: All possible swaps between two adjacent operations on the same machine. Used when the search doesn't find a better solution within `x` iterations.
2.  **Swap Critical**: Swaps between the first and last operations of each block within the critical path of each late operation. A critical path of a late operation $O$ is defined by all operations that prevent $O$ from being scheduled earlier. This is the default neighborhood when the others are not active.
3.  **CPLEX Relax**: Uses CPLEX to relax machine precedence for a block of early operations to find their optimal sequence. Triggered when the earliness penalty dominates the tardiness penalty by a certain factor.

## Evaluation of a Sequence

A "sequence" defines the order of operations on each machine, and the Tabu Search aims to find an optimal one. To calculate the objective value of a sequence, start times must be assigned to each operation. Three methods are available:

1.  **Schedule as Early as Possible**: Each operation is scheduled as early as its predecessors allow. This is effective when tardiness penalties dominate. If the `SHIFT_OPERS` flag is defined, an algorithm to delay operations without increasing penalties is executed to minimize earliness penalties.
2.  **Schedule Delaying when Advantageous**: Operations are scheduled from last to first. If delaying an operation (and its successors) reduces penalties, it is delayed until a deadline is reached or it hits another operation.
3.  **Schedule with CPLEX**: Uses CPLEX to find the optimal schedule for a given sequence. This is the most accurate but slowest method.

**Hybrid Method**: Combines methods 1 and 2. It uses "Early as Possible" when tardiness penalties are higher and "Delaying when Advantageous" when earliness penalties dominate.

## Best Known Configuration

Current best results were obtained with:
*   **Tabu tenure**: 29
*   **Scheduler type**: Hybrid (3)
*   **Iterations for Swap All**: 1
*   **Factor for CPLEX Relax**: 0.25

Example command:
```bash
./main instance name --tenure 29 --maxSecs 60 --schedulerType 3
```

## Publication

If you use this work or its results, please cite the following paper:

LELES, M. P. M.; FILHO, I. M. S.; BRUM, A. F.; ZUBARAN, T. K. **BUSCA TABU COM UMA NOVA VIZINHANÇA PARA O PROBLEMA JUST-IN-TIME JOB SHOP SCHEDULING**. In: XII Seminário de Iniciação Científica do IFNMG - 2025, 2025, Montes Claros. Anais eletrônicos... Montes Claros: IFNMG, 2025. 

Available at: [https://eventos.ifnmg.edu.br/sic2025/689bddaf7e2b2.pdf](https://eventos.ifnmg.edu.br/sic2025/689bddaf7e2b2.pdf)
