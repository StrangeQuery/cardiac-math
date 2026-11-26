# usqr-f

Computes x * x.

- Domain: x from 0 to 31 (32 inputs).
- Call: x in the accumulator, JMP 56.
- Returns: x * x in the accumulator.
- Overwrites: its own cells and cell 99.
- Cells: 43, 56-98.
- Instructions: worst 9, mean 9.0, from cell 56 to the return jump in cell 98.
- Algorithm: Table of squares, read by a CLA built from x.
- Sources: none found for this form.
- Correctness: proven, by exhaustive check on HEARTBURN of all 32 inputs, each called through the calling convention on one load, so on every call.
- Size: Not claimed smallest. Inline core 38 cells; as called here, 43.
- Speed: the instruction counts are exact, measured over the whole domain.
- Variant: The fast end of the frontier: a fixed 9 instructions. Not shipped: square-jump (36-cell core, worst 48), between the two.
- Error: Exact over the domain; outside it, x reads past the table.

Sources are listed in full in [../README.md](../README.md#sources).
