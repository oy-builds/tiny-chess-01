# tiny-chess

An original chess engine in at most 4,096 bytes of Python, written for
make.fun's tiny-chess-01. `main.py` is the whole engine: it carries its
source lzma-packed in one literal (the unpacker and payload count toward
the cap) and plays the env gauntlet at MAKEFUN_ENV_URL.

Chess: legal move generation (castling, en passant, promotions),
alpha-beta search with a transposition table, PVS, null-move pruning,
late-move reductions and check extensions, quiescence over captures,
promotions and check evasions, MVV-LVA/killer ordering, and an evaluation
of material, piece-square tables and the bishop pair. The search runs to
a fixed node budget (45,000) and never reads a clock, so a trial's replay
plays the same moves.

MIT licensed (see LICENSE).
