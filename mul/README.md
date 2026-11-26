# mul

Multiplication x * y and squaring x * x on CARDIAC's three-digit cells, one listing per variant.

| Variant | Computes | Range | Cells | Worst / mean | Destroys |
|---|---|---|---|---|---|
| umul-s | x * y | y >= 0, \|x * y\| <= 999, x of either sign | 17 (82-98) | 8,002 / 1,011.4 | y (cell 96) |
| **umul-m** | x * y | x, y >= 0, x * y <= 999 | 27 (72-98) | 267 / 52.8 | y (cell 96) |
| umul-f | x * y | x, y >= 0, x * y <= 999 | 39 (60-98) | 84 / 44.0 | y (cell 96) |
| usqr-s | x * x | x from 0 to 31 | 17 (82-98) | 289 / 149.5 | nothing |
| usqr-f | x * x | x from 0 to 31 | 43 (56-98) | 9 / 9.0 | nothing |

Default: **umul-m**. Its worst case is 30 times better than umul-s's for 10 more cells, and it doesn't depend on
the caller putting the smaller operand in y.

Instruction counts run from the routine's first cell to its return jump in cell 98, both included. The caller's
JMP and operand set-up are not counted. "Destroys" means caller-visible cells; every routine also overwrites cell 99
and its own work cells.

## When to choose another

- umul-s: the fewest cells, and the only one taking a negative x. It is fast when y is known to be small.
- umul-f: inner loops where speed matters and 39 cells are free.
- usqr-s: squares of 0..31 in 17 cells.
- usqr-f: squares of 0..31 in 9 instructions when 43 cells are free.
- A multiply with y = x also squares, but usqr-s is 10 cells smaller than umul-m; 31 is the largest x whose square
  fits.

## Trivial cases

No routine is needed for these.

- x * 10: `SFT 10`. x * 100: `SFT 20`.
- x * 2: `STO t`, `ADD t`. x * 3: `STO t`, `ADD t`, `ADD t`.
- x * k for a constant k: k - 1 cells of `ADD t` after `STO t`, or fewer with a shift. For example x * 9 is
  `STO t`, `SFT 10`, `SUB t`.

## Moving it

Rule: to move a routine down by k cells, subtract k from the address of every instruction that names one of the
routine's own cells, and from every data word marked "an instruction the routine copies and changes". Cells 00 and
99, constants and tables stay as they are.

| Variant | Address bases (data words that move) |
|---|---|
| umul-s | none |
| umul-m | none |
| umul-f | cell 92 (JMP 82) |
| usqr-s | none |
| usqr-f | cell 64 (CLA 65) |

## umul-s

- Range: y from 0 to 999 and x from -999 to 999 with \|x * y\| <= 999 (17,104 pairs). For y < 0 it returns 0.
  Products past 999 are not checked.
- Calling sample:

```
cell  contents   notes
 20     141      CLA 41   y into umul-s's operand cell 96
 21     696      STO 96
 22     140      CLA 40   x into the accumulator
 23     882      JMP 82   call umul-s
 24     642      STO 42   x * y, from the accumulator
```

- Answer: x * y in the accumulator.
- Destroys: y (cell 96), counted down to 0.
- Cells: 17, cells 82-98.
- Instructions: worst 8,002, mean 1,011.4.
- Algorithm: repeated addition. x is added once for each unit of y, so put the smaller operand in y.
- Sources: [TAYLOR].
- Correctness: exhaustive check on HEARTBURN of all 17,104 inputs, called through the calling convention one after
  another on one load, at the listed addresses and moved down to cell 20.
- Size: inline model 8 to 10 cells. No x * y of 7 cells or fewer exists without SFT (exhaustive search, 5.2 x 10^11
  branches); SFT is not covered by that search. The 10-cell loop here is the smallest known right on every call.
  As called here, 17 cells.
- Speed: exact, measured over the whole range.

## umul-m

- Range: x and y from 0 to 999 with x * y <= 999 (9,052 pairs). A negative operand returns 0. Products past 999
  are not checked.
- Calling sample:

```
cell  contents   notes
 20     141      CLA 41   y into umul-m's operand cell 96
 21     696      STO 96
 22     140      CLA 40   x into the accumulator
 23     872      JMP 72   call umul-m
 24     642      STO 42   x * y, from the accumulator
```

- Answer: x * y in the accumulator.
- Destroys: y (cell 96), which becomes the smaller operand and is counted down to 0.
- Cells: 27, cells 72-98.
- Instructions: worst 267, mean 52.8.
- Algorithm: repeated addition counting the smaller operand. x and y are exchanged when x < y, then x is added once
  for each unit of y. The loop turns at most 31 times, since the smaller operand of a product up to 999 is at most
  31.
- Sources: [TAYLOR] for repeated addition; none found for the exchange.
- Correctness: exhaustive check on HEARTBURN of all 9,052 inputs, called through the calling convention one after
  another on one load, at the listed addresses and moved down to cell 20.
- Size: not claimed smallest. Inline core 20 cells against the inline lower bound of 8 (no x * y of 7 cells or
  fewer without SFT, as for umul-s). As called here, 27 cells.
- Speed: exact, measured over the whole range.

## umul-f

- Range: x and y from 0 to 999 with x * y <= 999 (9,052 pairs). Negative operands and products past 999 are not
  checked.
- Calling sample:

```
cell  contents   notes
 20     141      CLA 41   y into umul-f's operand cell 96
 21     696      STO 96
 22     140      CLA 40   x into the accumulator
 23     860      JMP 60   call umul-f
 24     642      STO 42   x * y, from the accumulator
```

- Answer: x * y in the accumulator.
- Destroys: y (cell 96), divided down to 0.
- Cells: 39, cells 60-98.
- Instructions: worst 84, mean 44.0.
- Algorithm: multiplication by digit with a computed jump. For each digit d of y, from the last, it jumps into a row
  of nine `ADD x` at d cells from its end (the computed GO TO of FORTRAN, Duff's device), then takes y / 10 and 10x.
- Sources: none found for this form.
- Correctness: exhaustive check on HEARTBURN of all 9,052 inputs, called through the calling convention one after
  another on one load, at the listed addresses and moved down to cell 20.
- Size: not claimed smallest. Inline core 33 cells against the inline lower bound of 8 (as for umul-s). As called
  here, 39 cells.
- Speed: exact, measured over the whole range. The fastest of the multiplies measured.

## usqr-s

- Range: x from 0 to 31 (32 inputs). For x < 0 it returns 0. Past 31 the square does not fit in a cell.
- Calling sample:

```
cell  contents   notes
 20     140      CLA 40   x into the accumulator
 21     882      JMP 82   call usqr-s
 22     642      STO 42   x * x, from the accumulator
```

- Answer: x * x in the accumulator.
- Destroys: nothing but cell 99 and its own work cells.
- Cells: 17, cells 82-98.
- Instructions: worst 289, mean 149.5.
- Algorithm: sum of odd numbers. x * x = 1 + 3 + ... + (2x - 1), with x's cell counted down.
- Sources: [NICOMACHUS].
- Correctness: exhaustive check on HEARTBURN of all 32 inputs, called through the calling convention one after
  another on one load, at the listed addresses and moved down to cell 20.
- Size: inline model 8 to 11 cells. No square of 7 cells or fewer exists without SFT (exhaustive search, 2.9 x 10^11
  branches), and no straight-line routine of 10 or fewer with SFT. The 11-cell loop here is the smallest known right
  on every call. As called here, 17 cells.
- Speed: exact, measured over the whole range.

## usqr-f

- Range: x from 0 to 31 (32 inputs). Outside it, x reads past the table.
- Calling sample:

```
cell  contents   notes
 20     140      CLA 40   x into the accumulator
 21     856      JMP 56   call usqr-f
 22     642      STO 42   x * x, from the accumulator
```

- Answer: x * x in the accumulator.
- Destroys: nothing but cell 99 and its own work cells.
- Cells: 43, cells 56-98.
- Instructions: worst 9, mean 9.0 (every call takes 9).
- Algorithm: a table of squares, read by a `CLA` built from x.
- Sources: none found for this form.
- Correctness: exhaustive check on HEARTBURN of all 32 inputs, called through the calling convention one after
  another on one load, at the listed addresses and moved down to cell 20.
- Size: not claimed smallest. Inline core 38 cells against the inline lower bound of 8 (as for usqr-s). As called
  here, 43 cells.
- Speed: exact, measured over the whole range.

## Sources

- [TAYLOR] Taylor, A. E. IBM 1401, 1440 and 1460 programming and operating techniques. IBM Technical Information
  Exchange TIE 5-0021, 1964 (IBM internal). Section H-5: "the repetitive-addition method of multiplication".
- [NICOMACHUS] Nicomachus of Gerasa. Introduction to Arithmetic, about 100 AD: the squares as sums of the odd
  numbers (secondhand).
