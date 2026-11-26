# udvm-f

Computes x / y and x % y.

- Domain: x from 0 to 999, y from 1 to 999 (999,000 inputs); quotient truncated, remainder x - (x / y) y.
- Call: y in cell 79, x in the accumulator, JMP 61.
- Returns: the quotient x / y in the accumulator and the remainder x % y in cell 78.
- Overwrites: its own cells and cell 99; y in cell 79 is kept.
- Cells: 38, 61-98.
- Instructions: worst 275, mean 30.3, from cell 61 to the return jump in cell 98.
- Algorithm: Decimal long division, the quotient built by Horner's rule (q = 10q at each level, q + 1 at each subtraction).
- Sources: [IBM650]; [IBM1401].
- Correctness: proven, by exhaustive check on HEARTBURN of all 999,000 inputs, each called through the calling convention on one load, so on every call.
- Size: Not claimed smallest. Inline core 31 cells; as called here, 38.
- Speed: the instruction counts are exact, measured over the whole domain.
- Variant: The fast end of the frontier. Not shipped: divmod-ten (one level of tens, core 27 cells, worst 1,173), between the two.
- Error: Exact over the domain.

Sources are listed in full in [../README.md](../README.md#sources).
