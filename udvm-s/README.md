# udvm-s

Computes x / y and x % y.

- Domain: x from 0 to 999, y from 1 to 999 (999,000 inputs); quotient truncated, remainder x - (x / y) y.
- Call: y in cell 96, x in the accumulator, JMP 82.
- Returns: the quotient x / y in the accumulator and the remainder x % y in cell 95.
- Overwrites: its own cells and cell 99; y in cell 96 is kept.
- Cells: 17, 82-98.
- Instructions: worst 8,002, mean 36.3, from cell 82 to the return jump in cell 98.
- Algorithm: Repeated subtraction, counting: the count is the quotient, what is left the remainder.
- Sources: [ANDREE].
- Correctness: proven, by exhaustive check on HEARTBURN of all 999,000 inputs, each called through the calling convention on one load, so on every call.
- Size: Inline model: 8 to 10 cells. No x / y of 7 cells or fewer exists, with SFT or without (exhaustive search), and a divmod is also an x / y. The 10-cell loop here is the smallest known right on every call. As called here, 17 cells.
- Speed: the instruction counts are exact, measured over the whole domain.
- Variant: The small end of the frontier: 17 cells against udvm-f's 38; worst 8,002 instructions against 275. For the remainder alone, urem-s is smaller still (11 cells).
- Error: Exact over the domain.

Sources are listed in full in [../README.md](../README.md#sources).
