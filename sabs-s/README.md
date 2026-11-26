# sabs-s

Computes |x|.

- Domain: x from -999 to 999 (1,999 inputs).
- Call: x in the accumulator, JMP 89.
- Returns: |x| in the accumulator.
- Overwrites: its own cells and cell 99.
- Cells: 10, 89-98.
- Instructions: worst 8, mean 7.5, from cell 89 to the return jump in cell 98.
- Algorithm: Sign test; for x < 0, x - x - x = -x.
- Sources: none found for this form.
- Correctness: proven, by exhaustive check on HEARTBURN of all 1,999 inputs, each called through the calling convention on one load, so on every call.
- Size: Inline, |x| keeping x takes 5 cells and none of 4 exists without SFT; with SFT there is one of 4 that overwrites x (loop search). Its core is the 5-cell routine. As called here, 10 cells; no bound for the called form.
- Speed: the instruction counts are exact, measured over the whole domain.
- Variant: The only absolute value.
- Error: Exact over the domain.

Sources are listed in full in [../README.md](../README.md#sources).
