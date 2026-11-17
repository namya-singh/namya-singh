# Adversarial Maze Navigation

A full AI research project that pits an agent against a pursuer in a two-dimensional maze. The agent has to reach the goal before getting caught.

**Authors:** Namya Singh & Vikramaditya Sogani
**Course:** CS 4100 — Foundations of Artificial Intelligence, Northeastern University

---

## What This Project Does

You have an agent trying to get from a start position to a goal. A pursuer is also moving around the maze trying to catch the agent. We built nine different AI strategies for the agent and tested them all under four different conditions:

1. **Full visibility, fixed maze** — the agent can see everything and the walls do not move
2. **Full visibility, shifting walls** — the agent can see everything but walls randomly change position during the game
3. **Fog of war, fixed maze** — the agent can only see a small area around itself and has to figure out the rest as it goes
4. **Fog of war, shifting walls** — both limited vision and moving walls at the same time

The project also includes a visual game you can watch or play yourself, a tool for running many experiments automatically, and charts that show the results.

---

## Quick Start

### Install what you need

```bash
pip3 install pygame matplotlib
```

### Launch the visual game

```bash
python3 main.py --visual
```

### Run a terminal simulation comparing all algorithms

```bash
python3 main.py
```

### Run fog of war mode

```bash
python3 partial_game.py
```

### Run batch experiments

```bash
# Full visibility experiments only
python3 experiments.py --mode full

# Fog of war experiments only
python3 experiments.py --mode partial

# Both at the same time
python3 experiments.py --mode both

# Custom settings
python3 experiments.py --mode both --trials 5 --sizes 11 13 17 --densities 0.15 0.25 0.35

# Fog of war mode with custom visibility radius, shifting walls only
python3 experiments.py --mode partial --visibility 2 4 6 --dynamic-only
```

Results go to `experiment_outputs/run_<timestamp>/` as CSV files and PNG charts.

---

## Visual Game Controls

When you run `python3 main.py --visual`, these keys work:

| Key | What it does |
|-----|--------------|
| `SPACE` | Pause or resume the game |
| `R` | Start a new game with a fresh random maze |
| `F` | Turn fog of war on or off |
| `S` | Toggle split view so you can see both what the agent believes and the real maze |
| `D` | Turn shifting walls on or off |
| `+` or `-` | Speed up or slow down the simulation |
| `1` through `6` | Switch which algorithm the agent uses (see table below) |
| Arrow keys | Take over and move the agent yourself |
| `Q` or `ESC` | Quit |

### Which key maps to which algorithm

| Key | Algorithm |
|-----|-----------|
| `1` | Minimax |
| `2` | Alpha-Beta |
| `3` | Expectimax |
| `4` | LRTA* |
| `5` | Hill Climbing |
| `6` | Beam Search |

---

## File Guide

```
FAI_Final_Project/
|
|-- main.py               Start here. Runs terminal simulations or pass --visual for the game
|-- visual_game.py        The actual Pygame game window and all its controls
|-- experiments.py        Runs many games automatically and saves results and charts
|
|-- search.py             All the basic search algorithms:
|                           BFS, DFS, UCS, A*, Weighted A*, IDA*,
|                           Hill Climbing, Beam Search
|
|-- adversarial_search.py The game tree algorithms and pursuer AI:
|                           Minimax, Alpha-Beta, Expectimax, LRTA*
|                           PursuerAI with four difficulty levels
|
|-- game_state.py         Keeps track of whose turn it is and what the current game looks like
|-- maze.py               Generates static and shifting wall mazes
|-- node.py               Represents one step in a search path
|-- game_objects.py       Traps, power-ups, terrain types, and status effects
|
|-- partial_maze.py       Handles fog of war logic and what the agent currently believes
|-- partial_game.py       Runs fog of war games
|-- online_agent.py       The three fog of war agents: Online A*, Online LRTA*, Exploration
|
|-- test_partial.py       Testing utilities for fog of war mode
|
`-- experiment_outputs/   Auto-generated results folder
    `-- run_<timestamp>/
        |-- raw_results.csv
        |-- summary_results.csv
        `-- plots/
```

---

## The Algorithms

### Full Visibility Agents

| Algorithm | How it thinks | Will it always find a path? | Optimal path? | Speed |
|-----------|--------------|----------------------------|--------------|-------|
| Minimax | Looks several moves ahead assuming the pursuer plays perfectly | Yes, up to its lookahead depth | No | Slow |
| Alpha-Beta | Same as Minimax but skips branches that cannot change the outcome | Yes, up to its lookahead depth | No | Medium |
| Expectimax | Like Minimax but models the pursuer as sometimes random | Yes, up to its lookahead depth | No | Slow |
| LRTA* | Makes one move at a time, learns from experience | Yes, eventually | No | Very fast |
| Hill Climbing | Always steps toward what looks closest to the goal | No, can get stuck | No | Fast |
| Beam Search | Keeps the top few options open at each step | No, can get stuck | No | Fast |

The following algorithms are also implemented in `search.py` and used internally: BFS, DFS, UCS, A* with Manhattan or Euclidean distance, Weighted A*, and IDA*.

### Fog of War Agents

| Algorithm | What it does | Best used when |
|-----------|-------------|----------------|
| Online A* | Replans a full path every step based on what it currently knows | You want the highest win rate and have enough compute |
| Online LRTA* | Takes one step at a time using learned scores, never plans a full path | Speed matters and you want something close to LRTA* in full visibility mode |
| Exploration | Maps the maze first, then navigates to the goal | The goal location is completely unknown and mapping is worth the time |

### Pursuer Difficulty Levels

| Level | How the pursuer behaves |
|-------|------------------------|
| Random | Picks any legal move at random |
| Greedy | Always moves toward the agent by the shortest straight line |
| Beam | Uses beam search to navigate around walls toward the agent |
| A* | Runs a full pathfinding search to the agent every single turn |

---

## What the Maze Contains

### Cell Types

| Symbol | Type | Cost to enter | What happens |
|--------|------|--------------|--------------|
| (empty) | Open | 1 | Nothing |
| `#` | Wall | Cannot enter | Impassable |
| `T` | Trap | 1 | Agent cannot move for 3 turns |
| `P` | Power-up | 1 | Gives the agent a temporary ability |
| `~` | Mud | 3 | Slow to cross |
| `W` | Water | 2 | Slightly slow to cross |
| `+` | Road | 1 | Looks different but same speed as open |

### Power-Up Types

| Label | What it does | How long it lasts |
|-------|-------------|-------------------|
| `R` Reveal | Lifts fog of war so the agent can see the whole maze | 10 steps |
| `S` Speed | Agent takes two steps per turn instead of one | 8 steps |
| `X` Shield | Agent is immune to traps | 6 steps |
| `F` Freeze | Pursuer cannot move | 5 steps |

---

## Results at a Glance

These numbers come from the batch experiments in `experiment_outputs/`.

### Full Visibility

| Agent | Win Rate | Average Time per Game |
|-------|----------|-----------------------|
| Alpha-Beta | 40.6% | 9.8 ms |
| Minimax | 40.6% | 18.4 ms |
| Expectimax | 40.6% | 24.5 ms |
| Beam Search | 39.1% | 1.7 ms |
| LRTA* | 37.5% | 0.09 ms |
| Hill Climbing | 21.9% | 0.7 ms |

### Fog of War

| Agent | Win Rate | Average Time per Game |
|-------|----------|-----------------------|
| Online A* | 43.2% | 2.8 ms |
| Online LRTA* | 32.8% | 0.09 ms |
| Exploration | 31.3% | 0.3 ms |

The short version: Alpha-Beta wins as often as Minimax but runs twice as fast. LRTA* and Online LRTA* are the fastest options at under 0.1 ms per step. Online A* has the highest win rate overall.

---

## A Few Notes

- Run everything with `python3`, not `python`.
- Pygame is only needed for the visual game. All terminal and experiment modes work without it.
- Each time you run experiments a new timestamped folder is created, so old results are never overwritten.
- The `--visual` flag needs a screen attached. If you are on a server without a display, use terminal mode or experiments mode instead.
