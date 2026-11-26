# srem-s

Computes x % y, signed.

- Domain: x and y from -999 to 999, y not 0 (3,994,002 inputs); remainder with the sign of x (Rust's %).
- Call: y in cell 92, x in the accumulator, JMP 75.
- Returns: x % y in the accumulator.
- Overwrites: its own cells and cell 99, including y in cell 92 (made |y|).
- Cells: 24, 75-98.
- Instructions: worst 3,013, mean 22.7, from cell 75 to the return jump in cell 98.
- Algorithm: Repeated subtraction of |y| for x >= 0; for x < 0, repeated addition with the accumulator held at r - 1, so that TAC tests r + y <= 0.
- Sources: [ANDREE]; [IBM650]; [IBM1620].
- Correctness: proven, by exhaustive check on HEARTBURN of all 3,994,002 inputs, each called through the calling convention on one load, so on every call.
- Size: Inline model: at least 8 cells (exhaustive search). Core 18; as called here, 24. The gap from 8 to 18 is open.
- Speed: the instruction counts are exact, measured over the whole domain.
- Variant: The smallest signed remainder; sdvm-f gives it faster, in 71 cells. Not shipped: rem-signed-long (core 44 cells, worst 151).
- Error: Exact over the domain.

Sources are listed in full in [../README.md](../README.md#sources).
