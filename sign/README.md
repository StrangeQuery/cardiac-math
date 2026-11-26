# sign

Negation -x and absolute value |x| on CARDIAC's three-digit cells, one listing per routine.

| Variant | Computes | Range | Cells | Worst / mean | Destroys |
|---|---|---|---|---|---|
| **sneg-s** | -x | x from -999 to 999 | 8 | 7 / 7.0 | nothing |
| **sabs-s** | \|x\| | x from -999 to 999 | 10 | 8 / 7.5 | nothing |

Default: **sneg-s** and **sabs-s**, the only negate and the only absolute value.

Instruction counts run from the routine's first cell to its return jump in cell 98, both included. The caller's
JMP and set-up are not counted. "Destroys" means caller-visible cells; every routine also overwrites cell 99 and its
own work cells.

## When to choose another

- Inline code is smaller than either call. These routines are for programs that want every operation as a call.

## Trivial cases

No routine is needed for these.

- -x of a cell x: `CLA x`, `SUB x`, `SUB x` (3 cells).
- -x of the accumulator: `SFT 04` clears it, so `STO t`, `SFT 04`, `SUB t`.
- \|x\| of a cell x: `CLA x`, a `TAC` to the first of two `SUB x`, a `JMP` past them, `SUB x`, `SUB x` (5 cells).
- x - y and y - x need no negate: `CLA y`, `SUB x` gives y - x directly.

## Moving it

Rule: to move a routine down by k cells, subtract k from the address of every instruction that names one of the
routine's own cells. Cells 00 and 99 stay as they are. Neither routine holds an address base.

## sneg-s

- Range: x from -999 to 999 (1,999 inputs), every value a cell holds.
- Calling sample:

```
cell  contents   notes
 20     140      CLA 40   x into the accumulator
 21     891      JMP 91   call sneg-s
 22     642      STO 42   -x, from the accumulator
```

- Answer: -x in the accumulator.
- Destroys: nothing but cell 99 and its own work cells.
- Cells: 8.
- Instructions: worst 7, mean 7.0.
- Algorithm: `SFT 04` clears the accumulator, then `SUB x`.
- Sources: none found for this form.
- Correctness: exhaustive check on HEARTBURN of all 1,999 inputs, called through the calling convention one after
  another on one load, at the listed addresses and moved down to cell 20.
- Size: inline, -x of a cell takes 3 cells and no routine of 2 exists (straight-line search). With x in the
  accumulator, a routine right on every call takes 5, and none of 4 exists, with SFT or without (loop search). As
  called here, 8 cells; no bound for the called form.
- Speed: exact, measured over the whole range.

## sabs-s

- Range: x from -999 to 999 (1,999 inputs), every value a cell holds.
- Calling sample:

```
cell  contents   notes
 20     140      CLA 40   x into the accumulator
 21     889      JMP 89   call sabs-s
 22     642      STO 42   |x|, from the accumulator
```

- Answer: \|x\| in the accumulator.
- Destroys: nothing but cell 99 and its own work cells.
- Cells: 10.
- Instructions: worst 8, mean 7.5.
- Algorithm: a sign test; for x < 0, x - x - x = -x.
- Sources: none found for this form.
- Correctness: exhaustive check on HEARTBURN of all 1,999 inputs, called through the calling convention one after
  another on one load, at the listed addresses and moved down to cell 20.
- Size: inline, \|x\| keeping x takes 5 cells and none of 4 exists without SFT; with SFT there is one of 4 that
  overwrites x (loop search). Its core is the 5-cell routine. As called here, 10 cells; no bound for the called
  form.
- Speed: exact, measured over the whole range.

## Sources

None found for these forms.
