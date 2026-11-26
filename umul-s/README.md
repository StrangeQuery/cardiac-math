# umul-s

Computes x * y.

- Domain: y >= 0 and |x * y| <= 999, x of either sign (17,104 inputs).
- Call: y in cell 96, x in the accumulator, JMP 82.
- Returns: x * y in the accumulator.
- Overwrites: its own cells and cell 99, including y in cell 96 (counted down to 0).
- Cells: 17, 82-98.
- Instructions: worst 8,002, mean 1,011.4, from cell 82 to the return jump in cell 98.
- Algorithm: Repeated addition: x added once for each unit of y. Put the smaller operand in y.
- Sources: [TAYLOR].
- Correctness: proven, by exhaustive check on HEARTBURN of all 17,104 inputs, each called through the calling convention on one load, so on every call.
- Size: Inline model: 8 to 10 cells. No x * y of 7 cells or fewer exists without SFT (exhaustive search, 5.2 x 10^11 branches); SFT is not covered by that search. The 10-cell loop here is the smallest known right on every call. As called here, 17 cells.
- Speed: the instruction counts are exact, measured over the whole domain.
- Variant: The small end of the frontier: 17 cells against umul-f's 39; worst 8,002 instructions against 84.
- Error: Exact over the domain.

Sources are listed in full in [../README.md](../README.md#sources).
