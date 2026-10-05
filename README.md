# Tetris in Brainfuck

A terminal Tetris written entirely in Brainfuck: `tetris.bf`.
There is no generator, no macro language, and no helper code. Every instruction was written by hand.
Lines starting with `#` are comments, and they never contain a Brainfuck instruction character.

## Features

- 10x20 well with 2 hidden spawn rows, walls, and floor
- All 7 tetrominoes with SRS rotation (clockwise and counterclockwise)
- Full SRS wall kicks: 5 tests per rotation, separate table for I
- Left/right movement, soft drop (1 pt/row), hard drop (2 pts/row)
- Gravity driven by input ticks, with speed rising per level
- Collision detection against walls, floor, and stack
- Line clearing; scoring of 100/300/500/800 × level; a level up every 10 lines
- 7-bag randomizer, seeded by human timing
- Next-piece preview, hold (once per piece), ghost piece
- ANSI-colored terminal rendering; game-over detection

## Running

You need any Brainfuck interpreter with 8-bit wrapping cells and at least 4096 tape cells.
Brainfuck has no clock, so time passes when the program reads a byte `0` (a "tick").
The simplest way to supply ticks is to pipe NUL bytes in alongside your keystrokes:

```sh
stty -icanon -echo
{ while sleep 0.1; do printf '\0'; done & cat; } | bf tetris.bf
stty sane
```

Alternatively, if your interpreter reads with `read(2)` and treats end of input as "no change" or 0:

```sh
stty -icanon -echo min 0 time 1; bf tetris.bf; stty sane
```

## Controls

| Key | Action |
|---|---|
| `a` / `h` / ← | move left |
| `d` / `l` / → | move right |
| `s` / `j` / ↓ | soft drop |
| `w` / `x` / `k` / ↑ | rotate clockwise |
| `z` | rotate counterclockwise |
| space | hard drop |
| `c` | hold |
| `q` | quit |

## How it works

The header of `tetris.bf` documents the full tape layout. In short:

- **Board:** 30 rows × 16 columns. Each square is 8 tape cells: `F B T G` plus 4 scratch cells.
  `F` is 1 on every square, so `[<<<<<<<<]` always finds the way home.
- **State machine:** the main loop dispatches `STATE` through a 26-level nested decrement switch.
  There are no subroutines, so the shared "draw the candidate and test it" routine (`TRY`) is itself a state;
  callers leave their continuation in `RET`.
- **Rendering a piece:** row, column, and shape id are carried across the board to the piece's
  4x4 box corner. A 28-way switch then paints the four squares.
- **Collision:** a backward sweep carries a hit count square by square, so the count arrives at the globals.
- **Line clear:** a walker counts filled squares per row. On a full row, it drops a breadcrumb,
  pulls every square above down by one row, rebuilds the ceiling, and walks back to its breadcrumb.
- **Wall kicks:** a 64-case switch indexed by `32·isI + 16·direction + 4·rotation + test`.
