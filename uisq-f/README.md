# uisq-f

Computes floor(sqrt(x)).

- Domain: x from 0 to 999 (1,000 inputs).
- Call: x in the accumulator, JMP 50.
- Returns: floor(sqrt(x)) in the accumulator.
- Overwrites: its own cells and cell 99.
- Cells: 49, 50-98.
- Instructions: worst 100, mean 32.4, from cell 50 to the return jump in cell 98.
- Algorithm: Table start, then the sum of odd numbers: r0 = floor(sqrt(100a)) and r0^2 for a = x / 100 from two ten-cell tables, then at most nine odd numbers from 2 r0 + 1.
- Sources: [COLLINS]; [PRESTON].
- Correctness: proven, by exhaustive check on HEARTBURN of all 1,000 inputs, each called through the calling convention on one load, so on every call.
- Size: Not claimed smallest. Core 44 cells; as called here, 49.
- Speed: the instruction counts are exact, measured over the whole domain.
- Variant: The fast end of the frontier.
- Error: Exact over the domain.

Sources are listed in full in [../README.md](../README.md#sources).
