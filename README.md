[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/oy-builds/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/oy-builds/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

# Tiny Chess

A complete chess engine in a single Python file, built for make.fun's
tiny-chess-01 bounty. The entrypoint `main.py` is 4,071 bytes: the
LZMA-packed form of a readable source file, produced with the kit's own
`makefun compress` command. The source is developed alongside it in this
repository's working tree but is not part of the committed entry.

## How it plays

The engine talks to the competition env: it polls `POST /game` and
answers `POST /move` at `MAKEFUN_ENV_URL`. For every move it replays
the game from the start position (which also builds the repetition
history), then searches with a fixed budget of 110,000 nodes. There is
no clock and no randomness anywhere, so the same game always produces
the same moves and a replay matches the live run move for move.

## Design

- **Board**: a 0x88-style array of 192 cells; off-board cells hold a
  sentinel, so sliding-piece walks stop on their own. Moves are packed
  integers (from square, to square, promotion piece).
- **Search**: iterative-deepening negamax with alpha-beta and
  principal-variation search, a transposition table with
  exact/lower/upper bounds, null-move pruning, late-move reductions,
  MVV-LVA capture ordering, killer moves and a history heuristic for
  quiet moves, and a quiescence search over captures, promotions and
  check evasions.
- **Evaluation**: incremental material and piece-square tables for all
  six piece types (stored in a compact base-36 encoding), a bishop
  pair bonus, the king and pawn tables tapered toward the endgame by
  game phase, and dead-material draw detection. Positions are hashed incrementally with Zobrist keys,
  which also drive the repetition detection used by the search.
- **Rules**: full legal move generation — castling, en passant,
  promotions, check, checkmate and stalemate — with draws by
  repetition recognised inside the search.

## Verification

- Perft matches the known node counts exactly on five standard
  positions, including the start position (20 / 400 / 8902) and
  Kiwipete (48 / 2039 / 97862) at depths 1–3.
- On the kit's practice instances — six games each against the house
  ladder, from depth-2 up to a 500,000-node engine — this build went
  unbeaten in local runs played by the exact `main.py` bytes committed
  here: trial 0 scored 4 wins and 2 draws (Elo 1654, beating the
  100,000-node engine and drawing the 250,000- and 500,000-node ones),
  and trial 1 scored 6 wins (Elo 2112, the maximum), every win by
  checkmate. For scale, the house baseline rates about 1365 over the
  practice instances and the qualification floor is 1154. A typical
  move takes one to four seconds, well inside the 10-second limit.

## Layout

- `main.py` — the packed entrypoint, the only code the env runs.
- `LICENSE` — MIT.
- `.makefun/owner.json` — the entrant wallet for this bounty.
