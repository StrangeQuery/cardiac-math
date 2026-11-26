# urem-s

Computes x % y.

- Domain: x from 0 to 999, y from 1 to 999 (999,000 inputs).
- Call: y in cell 96, x in the accumulator, JMP 88.
- Returns: x % y in the accumulator.
- Overwrites: its own cells and cell 99; y in cell 96 is kept.
- Cells: 11, 88-98.
- Instructions: worst 3,005, mean 17.9, from cell 88 to the return jump in cell 98.
- Algorithm: Repeated subtraction with the running remainder in the accumulator, one y added back.
- Sources: [ANDREE].
- Correctness: proven, by exhaustive check on HEARTBURN of all 999,000 inputs, each called through the calling convention on one load, so on every call.
- Size: Proven smallest in the inline model: 5 cells, and no x % y of 4 cells or fewer exists, with SFT or without (exhaustive search). As called here, 11 cells; no bound for the called form.
- Speed: the instruction counts are exact, measured over the whole domain.
- Variant: The smallest remainder; udvm-s and udvm-f also give x % y.
- Error: Exact over the domain.

Sources are listed in full in [../README.md](../README.md#sources).
