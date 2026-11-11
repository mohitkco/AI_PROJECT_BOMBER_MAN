# BomberMan AI

A decision-making agent for a simplified Bomberman-style game. It combines a **danger map**, **A\* pathfinding** and **Minimax with alpha-beta pruning** to decide whether to move, wait or place a bomb. Built as an AI course project.

- Pure Python, **standard library only** (no installs needed)
- Single notebook: `bomberman_ai_fixed.ipynb`
- 13 asserted tests plus a self-play simulation

## How it works

```
Game state -> Danger map -> A* (escape cost) -> Evaluation -> Minimax -> chosen action
```

| Component | Role |
|-----------|------|
| `GameState`, `Bomb` | Plain data: grid, player positions, bombs, alive flags |
| `blast_cells`, `build_danger_map` | Which cells a bomb hits (stopped by walls) and a risk score per cell |
| `a_star`, `nearest_safe_cell` | Cheapest path where each step costs `1 + danger`; closest cell outside every blast |
| `legal_actions`, `apply_action`, `tick` | Game rules: moves, bomb placement, timers, explosions, chain reactions |
| `evaluate_state` | Scores a state (higher is better for the AI) |
| `minimax`, `choose_ai_action` | Looks ahead, assuming the enemy also plays well |

### Game rules

- Actions: `UP`, `DOWN`, `LEFT`, `RIGHT`, `WAIT`, `BOMB`
- A **round** is one AI move followed by one enemy move
- A bomb starts with `timer = 3` and ticks down after each round, so it explodes **6 plies** after being placed
- Blasts travel in 4 directions up to `radius` cells and stop at walls
- Anyone inside a blast dies; bombs caught in a blast explode too (chain reaction)

### Scoring (`evaluate_state`)

| Term | Effect |
|------|--------|
| AI dead / enemy dead / both dead | -10000 / +10000 / 0 |
| Base | +1000 |
| Danger at AI cell | -20 x danger |
| Danger at enemy cell | +15 x danger (rewards trapping) |
| Distance to enemy | -2 x abs(distance - 3) |
| Mobility | +3 x number of real moves |
| AI inside a blast zone | -15 x A\* path length to safety (or -500 if none) |

Because a bomb needs 6 plies to explode, the default search depth is **6**. A shallower search cannot see the explosion.

## Getting started

**Requirements:** Python 3.8+ and Jupyter (or Google Colab). No third-party packages.

1. Open `bomberman_ai_fixed.ipynb` in Jupyter or upload it to [Google Colab](https://colab.research.google.com).
2. Run all cells (**Runtime > Run all** in Colab).
3. The test section prints `PASS` for each test.

### Minimal usage

```python
grid = ["#######",
        "#.....#",
        "#.....#",
        "#.....#",
        "#######"]          # '#' = wall, '.' = floor

state = GameState(grid, ai=(1, 1), enemy=(3, 5))
action, score = choose_ai_action(state, depth=6)
print(action, score)                      # e.g. RIGHT 1007

state = apply_action(state, action, "ai")      # AI moves
state = apply_action(state, "WAIT", "enemy")   # enemy moves, bombs tick
```

Use `print_danger_map(state)` to see the danger values for any state.

## Tests

| Test | What it checks |
|------|----------------|
| 1 | A\* takes the safe 10-step detour instead of crossing the blast |
| 2, 3 | Nearest safe cell; `None` when the AI is already safe |
| 4, 5 | Timers explode and kill; chain reactions |
| 6, 7 | Walls block blasts and warnings; danger grows as the timer runs down |
| 8 | AI escapes an imminent bomb |
| 9 | AI traps the enemy in a dead end with `BOMB` (forced win, score 10000) |
| 10 | Legal actions are correct |
| 11 | Alpha-beta result equals plain minimax on 20 random states |
| 12 | Evaluation calls A\* and prefers distance 3 over adjacency |
| 13 | A\* edge cases: walls, start = goal, unreachable goal |
| 14 | Self-play vs a random enemy: **30 wins out of 30** games (about 5 s total) |

## Complexity

- **A\***: O((V + E) log V) with a binary heap
- **Minimax**: O(b^d), where b <= 6 actions and d = 6; alpha-beta best case O(b^(d/2))
- A depth-6 decision takes roughly 0.05 s

## Known limitations

- Simplified rules: no destructible boxes or power-ups; bombs do not block movement
- The AI assumes a rational opponent; a random enemy is easier than that
- Evaluation weights were chosen by hand and are not tuned
- Self-play against a random enemy is a sanity check, not a strong benchmark

## Possible improvements

- Destructible blocks and power-ups
- Tune the evaluation weights through self-play against stronger opponents
- Iterative deepening and move ordering to search deeper in the same time
- A graphical front end (for example, pygame)
