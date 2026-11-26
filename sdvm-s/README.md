# sdvm-s

Computes x / y and x % y, signed.

- Domain: x and y from -999 to 999, y not 0 (3,994,002 inputs); quotient truncated toward zero, remainder with the sign of x (Rust's / and %).
- Call: y in cell 89, x in the accumulator, JMP 52.
- Returns: the quotient x / y in the accumulator and the remainder x % y in cell 88.
- Overwrites: its own cells and cell 99, including y in cell 89 (negated when x < 0).
- Cells: 47, 52-98.
- Instructions: worst 8,021, mean 49.8, from cell 52 to the return jump in cell 98.
- Algorithm: Repeated subtraction, counting up for y > 0 and down for y < 0, after x and y are both negated when x < 0; the remainder given x's sign at the end.
- Sources: [ANDREE]; [IBM650]; [IBM1620].
- Correctness: proven, by exhaustive check on HEARTBURN of all 3,994,002 inputs, each called through the calling convention on one load, so on every call.
- Size: Inline model: at least 8 cells (no x / y or x % y of signed operands of 7 or fewer exists, with SFT or without; exhaustive search). Core 40; as called here, 47. The gap from 8 to 40 is open.
- Speed: the instruction counts are exact, measured over the whole domain.
- Variant: The small end of the frontier: 47 cells against sdvm-f's 71; worst 8,021 instructions against 304. For the remainder alone, srem-s is smaller (24 cells).
- Error: Exact over the domain.

Sources are listed in full in [../README.md](../README.md#sources).
