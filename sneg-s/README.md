# sneg-s

Computes -x.

- Domain: x from -999 to 999 (1,999 inputs).
- Call: x in the accumulator, JMP 91.
- Returns: -x in the accumulator.
- Overwrites: its own cells and cell 99.
- Cells: 8, 91-98.
- Instructions: worst 7, mean 7.0, from cell 91 to the return jump in cell 98.
- Algorithm: SFT 04 clears the accumulator, then SUB x.
- Sources: none found for this form.
- Correctness: proven, by exhaustive check on HEARTBURN of all 1,999 inputs, each called through the calling convention on one load, so on every call.
- Size: Inline, -x of a cell takes 3 cells and no routine of 2 exists (straight-line search). With x in the accumulator, a routine right on every call takes 5, and none of 4 exists, with SFT or without (loop search). As called here, 8 cells; no bound for the called form.
- Speed: the instruction counts are exact, measured over the whole domain.
- Variant: The only negate. Inline code is smaller; the routine is for callers that want every operation as a call.
- Error: Exact over the domain.

Sources are listed in full in [../README.md](../README.md#sources).
