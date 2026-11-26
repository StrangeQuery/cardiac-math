# uisr-f

Computes sqrt(x) to the nearest integer.

- Domain: x from 0 to 999 (1,000 inputs).
- Call: x in the accumulator, JMP 43.
- Returns: the nearest integer to sqrt(x) in the accumulator.
- Overwrites: its own cells and cell 99.
- Cells: 56, 43-98.
- Instructions: worst 105, mean 37.4, from cell 43 to the return jump in cell 98.
- Algorithm: Table start and odd numbers as uisq-f for r = floor(sqrt(x)), then r + 1 when x - r^2 > r, since (r + 1/2)^2 = r^2 + r + 1/4.
- Sources: [COLLINS].
- Correctness: proven, by exhaustive check on HEARTBURN of all 1,000 inputs, each called through the calling convention on one load, so on every call.
- Size: Not claimed smallest. Core 51 cells; as called here, 56.
- Speed: the instruction counts are exact, measured over the whole domain.
- Variant: The fast end of the frontier.
- Error: Exact over the domain.

Sources are listed in full in [../README.md](../README.md#sources).
