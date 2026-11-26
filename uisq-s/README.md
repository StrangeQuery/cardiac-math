# uisq-s

Computes floor(sqrt(x)).

- Domain: x from 0 to 999 (1,000 inputs).
- Call: x in the accumulator, JMP 81.
- Returns: floor(sqrt(x)) in the accumulator.
- Overwrites: its own cells and cell 99.
- Cells: 18, 81-98.
- Instructions: worst 322, mean 217.8, from cell 81 to the return jump in cell 98.
- Algorithm: Sum of odd numbers: count the odd numbers 1, 3, 5,... that can be taken from x; the count is the root.
- Sources: [NICOMACHUS]; [PRESTON]; [COCHRAN].
- Correctness: proven, by exhaustive check on HEARTBURN of all 1,000 inputs, each called through the calling convention on one load, so on every call.
- Size: Inline model: 8 to 13 cells. No routine of 7 cells or fewer exists, with SFT or without (exhaustive search). Core 13; as called here, 18.
- Speed: the instruction counts are exact, measured over the whole domain.
- Variant: The small end of the frontier: 18 cells against uisq-f's 49. Not shipped: isqrt-toepler (Toepler's method, core 24 cells, worst 137), between the two.
- Error: Exact over the domain.

Sources are listed in full in [../README.md](../README.md#sources).
