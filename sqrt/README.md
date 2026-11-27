# sqrt

Integer square roots floor(sqrt(x)) and sqrt(x) to the nearest integer on CARDIAC's three-digit cells, one listing
per variant.

| Variant | Computes | Range | Cells | Worst / mean | Destroys |
|---|---|---|---|---|---|
| **uisq-s** | floor(sqrt(x)) | x from 0 to 999 | 18 | 322 / 217.8 | nothing |
| uisq-f | floor(sqrt(x)) | x from 0 to 999 | 49 | 100 / 32.4 | nothing |
| uisr-s | sqrt(x) to the nearest integer | x from 0 to 999 | 20 | 302 / 203.5 | nothing |
| uisr-f | sqrt(x) to the nearest integer | x from 0 to 999 | 56 | 105 / 37.4 | nothing |

Default: **uisq-s**. Floor is the usual integer root, and at 18 cells it costs a third of the cells of uisq-f; its
worst case of 322 instructions is rarely what limits a program.

Instruction counts run from the routine's first cell to its return jump in cell 98, both included. The caller's
JMP and set-up are not counted. "Destroys" means caller-visible cells; every routine also overwrites cell 99 and its
own work cells.

## When to choose another

- uisq-f: roots in a loop where speed matters and 49 cells are free; mean 32 instructions against 218.
- uisr-s: the nearest root, for 2 cells more than uisq-s.
- uisr-f: the nearest root in a loop where speed matters and 56 cells are free.

## Trivial cases

No routine is needed for these.

- Testing x >= r * r for a known r: `CLA x`, `SUB` a cell holding r * r, `TAC`.
- Roots of a few known values: a table read by a built `CLA` (as uisq-f's tables) beats any loop.

## Moving it

Rule: to move a routine down by k cells, subtract k from the address of every instruction that names one of the
routine's own cells, and from every data word marked "an instruction the routine copies and changes". Cells 00 and
99, constants and tables stay as they are.

| Variant | Address bases (data words that move) |
|---|---|
| uisq-s | none |
| uisq-f | cell 65 (CLA 68) |
| uisr-s | none |
| uisr-f | cell 58 (CLA 62) |

## uisq-s

- Range: x from 0 to 999 (1,000 inputs). For x < 0 it returns 0.
- Calling sample:

```
cell  contents   notes
 20     140      CLA 40   x into the accumulator
 21     881      JMP 81   call uisq-s
 22     642      STO 42   floor(sqrt(x)), from the accumulator
```

- Answer: floor(sqrt(x)) in the accumulator.
- Destroys: nothing but cell 99 and its own work cells.
- Cells: 18.
- Instructions: worst 322, mean 217.8.
- Algorithm: sum of odd numbers. It counts the odd numbers 1, 3, 5, ... that can be taken from x; the count is the
  root.
- Sources: [NICOMACHUS]; [PRESTON]; [COCHRAN].
- Correctness: exhaustive check on HEARTBURN of all 1,000 inputs, called through the calling convention one after
  another on one load, at the listed addresses and moved down to cell 20.
- Size: inline model 8 to 13 cells. No routine of 7 cells or fewer exists, with SFT or without (exhaustive search).
  Core 13; as called here, 18 cells.
- Speed: exact, measured over the whole range.

## uisq-f

- Range: x from 0 to 999 (1,000 inputs). Negative x is not checked.
- Calling sample:

```
cell  contents   notes
 20     140      CLA 40   x into the accumulator
 21     850      JMP 50   call uisq-f
 22     642      STO 42   floor(sqrt(x)), from the accumulator
```

- Answer: floor(sqrt(x)) in the accumulator.
- Destroys: nothing but cell 99 and its own work cells.
- Cells: 49.
- Instructions: worst 100, mean 32.4.
- Algorithm: a table start, then the sum of odd numbers. r0 = floor(sqrt(100a)) and r0^2 for a = x / 100 come from
  two ten-cell tables, then at most nine odd numbers are taken from 2 r0 + 1.
- Sources: [COLLINS]; [PRESTON].
- Correctness: exhaustive check on HEARTBURN of all 1,000 inputs, called through the calling convention one after
  another on one load, at the listed addresses and moved down to cell 20.
- Size: not claimed smallest. Core 44 cells against the inline lower bound of 8 (as for uisq-s); as called here, 49
  cells.
- Speed: exact, measured over the whole range.

## uisr-s

- Range: x from 0 to 999 (1,000 inputs). For x < 0 it returns 0.
- Calling sample:

```
cell  contents   notes
 20     140      CLA 40   x into the accumulator
 21     879      JMP 79   call uisr-s
 22     642      STO 42   sqrt(x) to the nearest integer, from the accumulator
```

- Answer: the nearest integer to sqrt(x) in the accumulator.
- Destroys: nothing but cell 99 and its own work cells.
- Cells: 20.
- Instructions: worst 302, mean 203.5.
- Algorithm: sum of even numbers. The nearest root counts the k >= 1 with x >= k^2 - k + 1, whose differences are
  2, 4, 6, ..., so it is the odd-number loop with 2r for 2r + 1, from x - 1. No ties occur.
- Sources: none found for this form.
- Correctness: exhaustive check on HEARTBURN of all 1,000 inputs, called through the calling convention one after
  another on one load, at the listed addresses and moved down to cell 20.
- Size: inline model 8 to 15 cells. No routine of 7 cells or fewer exists, with SFT or without (exhaustive search).
  Core 15; as called here, 20 cells.
- Speed: exact, measured over the whole range.

## uisr-f

- Range: x from 0 to 999 (1,000 inputs). Negative x is not checked.
- Calling sample:

```
cell  contents   notes
 20     140      CLA 40   x into the accumulator
 21     843      JMP 43   call uisr-f
 22     642      STO 42   sqrt(x) to the nearest integer, from the accumulator
```

- Answer: the nearest integer to sqrt(x) in the accumulator.
- Destroys: nothing but cell 99 and its own work cells.
- Cells: 56.
- Instructions: worst 105, mean 37.4.
- Algorithm: the table start and odd numbers of uisq-f for r = floor(sqrt(x)), then r + 1 when x - r^2 > r, since
  (r + 1/2)^2 = r^2 + r + 1/4.
- Sources: [COLLINS].
- Correctness: exhaustive check on HEARTBURN of all 1,000 inputs, called through the calling convention one after
  another on one load, at the listed addresses and moved down to cell 20.
- Size: not claimed smallest. Core 51 cells against the inline lower bound of 8 (as for uisr-s); as called here, 56
  cells.
- Speed: exact, measured over the whole range.

## Sources

- [NICOMACHUS] Nicomachus of Gerasa. Introduction to Arithmetic, about 100 AD: the squares as sums of the odd
  numbers (secondhand).
- [PRESTON] IBM. IBM 1401 Programs, catalog C20-1601-5, 1962. Program 1401-03.0.014 (B. Preston): "square root
  subroutine using odd integer method".
- [COCHRAN] Cochran, D. S. Internal programming of the 9100A calculator. Hewlett-Packard Journal 20(1), September
  1968, pp. 14-16. p. 15: the square root "is done by pseudo division" from the sum of the odd numbers.
- [COLLINS] Collins, G. E. Square root subroutine. IBM 650 Library Program Abstract, 1956. In Additional Abstracts
  of IBM 650 Programs, March 1957: a table look-up followed by two Newton iterations.
