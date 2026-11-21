# Tiny Chess

A complete chess engine in one Python file, built for make.fun's
tiny-chess-01 bounty. The entrypoint `main.py` is 4,071 bytes: the
LZMA-packed form of a readable source, produced with the kit's own
`makefun compress`. The source lives in this repository's working
tree but is not part of the committed entry.

## How it plays

The engine polls `POST /game` and answers `POST /move` at
`MAKEFUN_ENV_URL`. For each move it replays the game from the start
(which also builds the repetition history), then searches with a
fixed budget of 110,000 nodes. No clock, no randomness: the same
game always produces the same moves.

## Design

- **Board**: 0x88-style array of 192 cells with an off-board
  sentinel; moves are packed integers.
- **Search**: iterative-deepening negamax with alpha-beta and PVS, a
  transposition table, null-move pruning, late-move reductions,
  MVV-LVA ordering, killer moves, a history heuristic, and
  quiescence over captures, promotions and evasions.
- **Evaluation**: incremental material and piece-square tables for
  all six piece types (compact base-36 encoding), a bishop-pair
  bonus, king and pawn tables tapered by game phase, dead-material
  draw detection, Zobrist hashing.
- **Rules**: full legal move generation — castling, en passant,
  promotions, checkmate and stalemate — with repetition recognised
  inside the search.

## Verification

- Perft matches known counts exactly on five positions, including
  startpos (20 / 400 / 8902) and Kiwipete (48 / 2039 / 97862) at
  depths 1–3.
- Practice, played by the exact committed `main.py` bytes: trial 0
  went 4 wins, 2 draws (Elo 1654 — beat the 100,000-node engine, drew
  the 250k and 500k ones); trial 1 went 6 wins (Elo 2112, the
  maximum), every win by checkmate. The house baseline rates about
  1365; the qualification floor is 1154. A typical move takes one to
  four seconds, inside the 10-second limit.

## Layout

- `main.py` — the packed entrypoint, the only code the env runs.
- `LICENSE` — MIT.
- `.makefun/owner.json` — the entrant wallet for this bounty.
