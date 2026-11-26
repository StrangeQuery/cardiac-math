# uisr-s

Computes sqrt(x) to the nearest integer.

- Domain: x from 0 to 999 (1,000 inputs).
- Call: x in the accumulator, JMP 79.
- Returns: the nearest integer to sqrt(x) in the accumulator.
- Overwrites: its own cells and cell 99.
- Cells: 20, 79-98.
- Instructions: worst 302, mean 203.5, from cell 79 to the return jump in cell 98.
- Algorithm: Sum of even numbers: the nearest root counts the k >= 1 with x >= k^2 - k + 1, whose differences are 2, 4, 6,..., so it is the odd-number loop with 2r for 2r + 1, from x - 1. No ties occur.
- Sources: none found for this form.
- Correctness: proven, by exhaustive check on HEARTBURN of all 1,000 inputs, each called through the calling convention on one load, so on every call.
- Size: Inline model: 7 to 15 cells. No routine of 6 cells or fewer exists, with SFT or without (exhaustive search). Core 15; as called here, 20.
- Speed: the instruction counts are exact, measured over the whole domain.
- Variant: The small end of the frontier: 20 cells against uisr-f's 56. Not shipped: isqrt-near-toepler (core 30 cells, worst 141).
- Error: Exact over the domain.

Sources are listed in full in [../README.md](../README.md#sources).
