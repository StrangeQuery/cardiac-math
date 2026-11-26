# sdvm-f

Computes x / y and x % y, signed.

- Domain: x and y from -999 to 999, y not 0 (3,994,002 inputs); quotient truncated toward zero, remainder with the sign of x (Rust's / and %).
- Call: y in cell 54, x in the accumulator, JMP 28.
- Returns: the quotient x / y in the accumulator and the remainder x % y in cell 53.
- Overwrites: its own cells and cell 99, including y in cell 54 (made |y|).
- Cells: 71, 28-98.
- Instructions: worst 304, mean 50.3, from cell 28 to the return jump in cell 98.
- Algorithm: Decimal long division of the magnitudes, the quotient built by Horner's rule; the signs put back.
- Sources: [IBM650]; [IBM1401]; [IBM1620].
- Correctness: proven, by exhaustive check on HEARTBURN of all 3,994,002 inputs, each called through the calling convention on one load, so on every call.
- Size: Not claimed smallest. Core 65 cells; as called here, 71.
- Speed: the instruction counts are exact, measured over the whole domain.
- Variant: The fast end of the frontier.
- Error: Exact over the domain.

Sources are listed in full in [../README.md](../README.md#sources).
