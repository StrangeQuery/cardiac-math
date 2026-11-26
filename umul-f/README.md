# umul-f

Computes x * y.

- Domain: x, y >= 0 and x * y <= 999 (9,052 inputs).
- Call: y in cell 96, x in the accumulator, JMP 60.
- Returns: x * y in the accumulator.
- Overwrites: its own cells and cell 99, including y in cell 96 (divided down to 0).
- Cells: 39, 60-98.
- Instructions: worst 84, mean 44.0, from cell 60 to the return jump in cell 98.
- Algorithm: Multiplication by digit with a computed jump: for each digit d of y, from the last, a jump into a row of nine ADD x at d cells from its end (the computed GO TO of FORTRAN, Duff's device), then y / 10 and 10x.
- Sources: none found for this form.
- Correctness: proven, by exhaustive check on HEARTBURN of all 9,052 inputs, each called through the calling convention on one load, so on every call.
- Size: Not claimed smallest. Inline core 33 cells; as called here, 39.
- Speed: the instruction counts are exact, measured over the whole domain.
- Variant: The fast end of the frontier of the multiplies measured. Not shipped: mul-swap (repeated addition counting the smaller operand, core 20 cells, worst 262), between the two.
- Error: Exact over the domain.

Sources are listed in full in [../README.md](../README.md#sources).
