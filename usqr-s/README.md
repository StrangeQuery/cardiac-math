# usqr-s

Computes x * x.

- Domain: x from 0 to 31 (32 inputs).
- Call: x in the accumulator, JMP 82.
- Returns: x * x in the accumulator.
- Overwrites: its own cells and cell 99.
- Cells: 17, 82-98.
- Instructions: worst 289, mean 149.5, from cell 82 to the return jump in cell 98.
- Algorithm: Sum of odd numbers: x * x = 1 + 3 +... + (2x - 1), x's cell counted down.
- Sources: [NICOMACHUS].
- Correctness: proven, by exhaustive check on HEARTBURN of all 32 inputs, each called through the calling convention on one load, so on every call.
- Size: Inline model: 8 to 11 cells. No square of 7 cells or fewer exists without SFT (exhaustive search, 2.9 x 10^11 branches), and no straight-line routine of 10 or fewer with SFT. The 11-cell loop here is the smallest known right on every call. As called here, 17 cells.
- Speed: the instruction counts are exact, measured over the whole domain.
- Variant: The small end of the frontier: 17 cells against usqr-f's 43; worst 289 instructions against 9.
- Error: Exact over the domain.

Sources are listed in full in [../README.md](../README.md#sources).
