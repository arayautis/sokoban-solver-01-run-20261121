# sokoban-solver-01-run-20261121

[![make.fun · sokoban-solver-01-run-20261121](https://make.fun/badge/sokoban-solver-01-run-20261121.svg)](https://make.fun/bounty/sokoban-solver-01-run-20261121?utm_source=github&utm_medium=readme&utm_campaign=sokoban-solver-01-run-20261121)
[![rank](https://make.fun/badge/sokoban-solver-01-run-20261121/arayautis/sokoban-solver-01-run-20261121/rank.svg)](https://make.fun/bounty/sokoban-solver-01-run-20261121/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=sokoban-solver-01-run-20261121)
[![reward](https://make.fun/badge/sokoban-solver-01-run-20261121/arayautis/sokoban-solver-01-run-20261121/reward.svg)](https://make.fun/bounty/sokoban-solver-01-run-20261121?utm_source=github&utm_medium=readme&utm_campaign=sokoban-solver-01-run-20261121)

A push-optimal Sokoban solver for make.fun's "Build the best Sokoban solver, 21 November 2026" bounty. One program (`main.ts`, Node 22, no dependencies) answers all 5 instances of a set in about a second, in the fewest pushes possible.

## How it works

Every level the generator draws is a hallway with independent rooms off it: a box pushed into the hallway can never come back, so each room's boxes stay in their room and the level's push-optimum is the sum of the rooms' optima. The solver exploits that:

1. **Split into rooms.** The dead squares of the player's row (the hallway — a box there reaches no goal) are cut out of the floor; each remaining part that holds a box is one room.
2. **Breadth-first search over pushes, per room.** A state is the boxes' squares plus the player's reachable region (named by its lowest square). Expanding in order of pushes made makes the first solved state push-optimal. Two prunings keep the search small:
   - *dead squares* — a square from which no push (ignoring the other boxes) can ever bring a box onto a goal, found by pulling boxes back from the goals over open floor;
   - *frozen 2×2 blocks* — four squares of walls and boxes holding a box off its goal, which no push can break apart.
3. **Walk the player between rooms** along the hallway; the other rooms' boxes stand as walls while one room is solved.

The search budget is counted in states, never in seconds — the wall clock is never read — so two runs on one set write byte-identical answers.

## Results

| Set | Score | House baseline (plain-astar 1.0.0) |
| --- | --- | --- |
| Practice (kit) | **1,000,000** — all 5 instances at exactly P\* pushes (13, 16, 29, 47, 60) | 200,000 (only i001 solved) |
| 20 freshly drawn sets (kit generator, random seeds) | **1,000,000 on all 20**, worst run 1.5 s of the 60 s budget | — |

## Run it

```sh
MAKEFUN_INPUT_DIR=practice/input MAKEFUN_OUTPUT_DIR=out node main.ts
```

Reads `/input/<id>/level.txt` (or `$MAKEFUN_INPUT_DIR`) and writes `/output/<id>/solution.txt` — the player's moves in LURD, uppercase for pushes — for every instance. Deterministic: two runs on the same set produce the same bytes.
