# div

Division and remainder for CARDIAC: general routines for a run-time divisor, called, and straight-line routines for
fixed divisors, inlined.

## General routines

The calling convention is the one in the top-level README: y in the routine's y cell, x in the accumulator, JMP to
the first cell. Instruction counts run from the routine's first cell to its return jump, both included, worst and
mean over the whole domain; the caller's set-up and JMP are not counted.

| Routine | Computes | Range | Cells | Worst / mean | Destroys |
|---|---|---|---|---|---|
| [udvm-s](udvm-s.cardiac) | x / y, x % y | x 0..999, y 1..999 | 17 (82-98) | 8,002 / 36.3 | own cells, 99 |
| **[udvm-m](udvm-m.cardiac)** | **x / y, x % y** | **x 0..999, y 1..999** | **34 (65-98)** | **1,178 / 28.6** | **own cells, 99** |
| [udvm-f](udvm-f.cardiac) | x / y, x % y | x 0..999, y 1..999 | 38 (61-98) | 275 / 30.3 | own cells, 99 |
| [urem-s](urem-s.cardiac) | x % y | x 0..999, y 1..999 | 11 (88-98) | 3,005 / 17.9 | own cells, 99 |
| [sdvm-s](sdvm-s.cardiac) | x / y, x % y | x, y -999..999, y not 0 | 47 (52-98) | 8,021 / 49.8 | own cells, 99, y |
| [sdvm-f](sdvm-f.cardiac) | x / y, x % y | x, y -999..999, y not 0 | 71 (28-98) | 304 / 50.3 | own cells, 99, y |
| [srem-s](srem-s.cardiac) | x % y | x, y -999..999, y not 0 | 24 (75-98) | 3,013 / 22.7 | own cells, 99, y |

The default is **udvm-m**: the lowest mean of the three divmods (28.6), a worst case under a seventh of udvm-s's,
and 4 cells fewer than udvm-f.

All of them truncate the quotient toward zero and give the remainder the dividend's sign, as Rust's `/` and `%` do.
"y" under Destroys means the routine changes its y cell; the others keep y, so a program that always divides by the
same y can store it once.

### When to choose another

- udvm-s: smallest divmod (17 cells), when speed doesn't matter; its worst case is 8,002 instructions.
- udvm-f: inner loops; worst case 275.
- urem-s: only the remainder is needed; its 5-cell core is proven smallest.
- sdvm-s, sdvm-f, srem-s: signed operands.
- A constant routine below, whenever the divisor is a fixed constant in the table.
- SFT, for powers of ten (below).

## Constant routines

Straight-line code for x / d or x % d with d fixed, to put in the program where the result is needed: no call, no
return. Each listing is at cells 10 and up, with x in cell 90 and, where one is used, a scratch cell t0 in cell 91.
To use one, copy its instructions into the program and change each 90 to the program's cell for x, and each 91 to
a free cell; the program goes on with the result in the accumulator. The code has no jumps and no addresses of its
own, so it works at any place. It overwrites the accumulator and t0 and keeps x.

Example: x / 5 of the program's cell 31, at cells 20-22:

```
cell  contents   notes
 20     131      CLA 31   sdiv5, its 90 changed to 31
 21     231      ADD 31
 22     401      SFT 01
 23     632      STO 32   x / 5, from the accumulator
```

The prefix gives the domain over which the routine is exact: `s` for x from -999 to 999 (quotient truncated toward
zero, remainder with the sign of x), `u` for x from 0 to 999 only. Cells count the instructions and t0, not x's
cell; instructions are what runs, always the same. "Smallest" is relative to the straight-line model: CLA, ADD,
SUB, SFT and STO on x, cell 00 and scratch cells, every scratch cell counted. "Proven" means an exhaustive search
finds no shorter routine in that model; otherwise the column gives the proven lower bound.

The comparison is with calling udvm-m with y = d, for x from 0 to 999. A call costs `CLA K; STO 88; CLA x; JMP 65`,
four cells and four instructions, and a cell K holding d; a remainder costs one more, `CLA 87`. The udvm-m column
gives the instructions of the whole call, worst / mean, and its cells with udvm-m already in the program and,
in brackets, with it brought in for this (34 cells more). For signed x udvm-m does not apply, and sdvm-s or sdvm-f
cost more.

| Routine | Computes | Domain | Cells | Instructions | Smallest | Through udvm-m: cells, instructions | Shipped because |
|---|---|---|---|---|---|---|---|
| [sdiv2](sdiv2.cardiac) | x / 2 | -999..999 | 6 | 6 | proven | 5 (39); 632 / 326.5 | 105x faster at worst, 54x at mean |
| [sdiv3](sdiv3.cardiac) | x / 3 | -999..999 | 17 | 17 | at least 11 | 5 (39); 445 / 234.6 | 26x faster at worst, 14x at mean |
| [sdiv4](sdiv4.cardiac) | x / 4 | -999..999 | 9 | 9 | proven | 5 (39); 357 / 189.0 | 40x faster at worst, 21x at mean |
| [sdiv5](sdiv5.cardiac) | x / 5 | -999..999 | 3 | 3 | proven | 5 (39); 302 / 161.5 | 101x faster at worst, 54x at mean; smaller |
| [sdiv6](sdiv6.cardiac) | x / 6 | -999..999 | 17 | 17 | at least 11 | 5 (39); 258 / 142.7 | 15x faster at worst, 8x at mean |
| [sdiv7](sdiv7.cardiac) | x / 7 | -999..999 | 11 | 11 | proven | 5 (39); 236 / 129.6 | 21x faster at worst, 12x at mean |
| [sdiv8](sdiv8.cardiac) | x / 8 | -999..999 | 11 | 11 | proven | 5 (39); 214 / 119.6 | 19x faster at worst, 11x at mean |
| [sdiv9](sdiv9.cardiac) | x / 9 | -999..999 | 9 | 9 | proven | 5 (39); 203 / 112.3 | 23x faster at worst, 12x at mean |
| [sdiv11](sdiv11.cardiac) | x / 11 | -999..999 | 8 | 8 | proven | 5 (39); 181 / 101.2 | 23x faster at worst, 13x at mean |
| [sdiv12](sdiv12.cardiac) | x / 12 | -999..999 | 18 | 18 | at least 11 | 5 (39); 170 / 96.4 | 9x faster at worst, 5x at mean |
| [udiv12](udiv12.cardiac) | x / 12 | 0..999 | 14 | 14 | at least 11 | 5 (39); 170 / 96.4 | 12x faster at worst, 7x at mean; 4 cells under sdiv12 |
| [sdiv25](sdiv25.cardiac) | x / 25 | -999..999 | 5 | 5 | proven | 5 (39); 126 / 73.5 | 25x faster at worst, 15x at mean; no larger |
| [srem2](srem2.cardiac) | x % 2 | -999..999 | 10 | 9 | proven | 6 (40); 633 / 327.5 | 70x faster at worst, 36x at mean |
| [srem3](srem3.cardiac) | x % 3 | -999..999 | 20 | 19 | at least 11 | 6 (40); 446 / 235.6 | 23x faster at worst, 12x at mean |
| [urem3](urem3.cardiac) | x % 3 | 0..999 | 16 | 15 | at least 11 | 6 (40); 446 / 235.6 | 30x faster at worst, 16x at mean; 4 cells under srem3 |
| [srem4](srem4.cardiac) | x % 4 | -999..999 | 15 | 14 | at least 11 | 6 (40); 358 / 190.0 | 26x faster at worst, 14x at mean |
| [srem5](srem5.cardiac) | x % 5 | -999..999 | 10 | 9 | proven | 6 (40); 303 / 162.5 | 34x faster at worst, 18x at mean |
| [srem9](srem9.cardiac) | x % 9 | -999..999 | 10 | 10 | proven | 6 (40); 204 / 113.3 | 20x faster at worst, 11x at mean |
| [srem12](srem12.cardiac) | x % 12 | -999..999 | 26 | 25 | at least 11 | 6 (40); 171 / 97.4 | 7x faster at worst, 4x at mean |
| [urem12](urem12.cardiac) | x % 12 | 0..999 | 22 | 21 | at least 11 | 6 (40); 171 / 97.4 | 8x faster at worst, 5x at mean; 4 cells under srem12 |

The udvm-m figures per call are 4 or 5 instructions of set-up and these counts inside udvm-m (worst / mean, x from 0
to 999): y = 2: 628 / 322.5; 3: 441 / 230.6; 4: 353 / 185.0; 5: 298 / 157.5; 6: 254 / 138.7; 7: 232 / 125.6;
8: 210 / 115.6; 9: 199 / 108.3; 11: 177 / 97.2; 12: 166 / 92.4; 25: 122 / 69.5. When the program keeps y = d in
cell 88 from the start, each call shrinks to `CLA x; JMP 65`, two cells and two instructions.

| Routine | Algorithm |
|---|---|
| sdiv2 | 5x / 10 |
| sdiv3 | 3334x / 10^4 by Horner's rule: a digit of 3334 in additions of x, then a right shift, last digit first |
| sdiv4 | 5x / 10 = x / 2, plus 2x, shifted again: 25x / 100 |
| sdiv5 | 2x / 10 |
| sdiv6 | 1667x / 10^4 as -333x / 1000 + 2x, shifted; the truncations cancel |
| sdiv7 | 143x / 1000 by Horner's rule |
| sdiv8 | 125x / 1000 by Horner's rule |
| sdiv9 | 1112x / 10^4 by Horner's rule |
| sdiv11 | 909x / 10^4 by signed digits: 9x / 100, then -x, then +x, a shift after each |
| sdiv12 | 834x / 10^4 by Horner's rule |
| udiv12 | (833x - 100) / 10^4 by signed digits, the constant from cell 00 (multiply-add form, x >= 0 only) |
| sdiv25 | 4x / 100 |
| srem2 | 5x; SFT 31 keeps its last digit (5 or 0) times 100; doubled and divided by 1000 |
| srem3 | 3334x modulo 10^4 falls in one band per remainder; the band read by a shift, times 4 and SFT 03 |
| urem3 | v = 333(x + 1) modulo 1000 (each STO keeps three digits), then 3v / 1000 |
| srem4 | 25x; SFT 22 keeps its last two digits, 25 times the remainder; times 4, divided by 100 |
| srem5 | 2x; SFT 31 keeps its last digit, twice the remainder, times 100; times 5, divided by 1000 |
| srem9 | 1111x modulo 10^4, whose thousands digit is the remainder; ADD x, SUB x corrects remainder 0 through the accumulator's wrap |
| srem12 | q = x / 12 as sdiv12, then x - 12q as -12q + x |
| urem12 | q = x / 12 as udiv12, then x - 12q as -12q + x |

The multipliers N / 10^k meet the exactness condition of [LEMIRE21] with modulus 10^k. A chain whose digits are all
positive computes trunc(N x / 10^k) with no error from the intermediate shifts, since every term has the sign of x;
the chains with negative digits (sdiv6, sdiv11, udiv12) are exact by the exhaustive check alone. On a sign-magnitude
machine a routine that uses no constant is an odd function, so it holds for negative x as well. The direct
remainders are the decimal form of [LEMIRE19]; the multiply-add forms (udiv12, urem3) are those of [ROBISON];
[GRANLUND] is the binary method behind all of these.

Status: each routine was run on HEARTBURN at its listed cells for every x of its domain (1,999 for s, 1,000 for u),
from a different accumulator each time, checking the result, that it ran exactly its instructions, and that no cell
but t0 changed; then again with the code moved to cell 40 and x and t0 to cells 70 and 71. The u routines are wrong
for some negative x (udiv12 and urem12 for 41 of them, urem3 for all). Size bounds come from an exhaustive search
of straight-line routines up to ten cells: it finds those marked proven and rules out every routine of ten cells
or fewer for 3, 6, 7, 8 and 12 (division) and 3, 4 and 12 (remainder); sdiv7 and sdiv8, at eleven, meet that bound.

Every candidate passed the test, so none was dropped: the smallest gain, srem12's, is 4 times at the mean and 7 at
worst.

### Powers of ten

One instruction, so no routine. SFT acts on the magnitude and keeps the sign, so the quotients truncate toward zero
and the remainders take the dividend's sign. Each was checked on HEARTBURN for every accumulator value from -9999 to
9999, and in the `CLA x` form for every x from -999 to 999.

- x / 10, x / 100, x / 1000: SFT 01, SFT 02, SFT 03 (`CLA x; SFT 01` for a cell).
- x % 10: SFT 33. x % 100: SFT 22.
- x % 1000 of the accumulator is free: STO keeps only the low three digits and the sign, so storing the accumulator
  already reduces it modulo 1000. A cell already holds x % 1000, and x / 1000 of a cell is 0.

## udvm-s

- Range: x from 0 to 999, y from 1 to 999 (999,000 inputs); quotient truncated, remainder x - (x / y) y.
- Calling:

```
cell  contents   notes
 10     130      CLA 30   y
 11     696      STO 96   into udvm-s's y cell
 12     131      CLA 31   x into the accumulator
 13     882      JMP 82   call udvm-s
 14     632      STO 32   the quotient, from the accumulator
 15     195      CLA 95   the remainder, from cell 95
```

  Quotient in the accumulator, remainder in cell 95.
- Destroys: the accumulator, its cells 82-98 except y in cell 96, and cell 99.
- Cells: 17, 82-98.
- Instructions: worst 8,002, mean 36.3.
- Algorithm: repeated subtraction, counting: the count is the quotient, what is left the remainder.
- Sources: [ANDREE].
- Status: correct by exhaustive check on HEARTBURN of all 999,000 inputs, called one after another on one load,
  and again after moving it to start at cell 20. Size: inline model 8 to 10 cells; no x / y of 7 cells or fewer
  exists, with SFT or without (exhaustive search), and a divmod is also an x / y. The 10-cell loop here is the
  smallest known right on every call; as called, 17 cells. Speed: exact.
- Moving it: subtract k from the address of every instruction that names a cell in 82-98; 00 and 99 stay.
  Address bases: none.

## udvm-m

- Range: x from 0 to 999, y from 1 to 999 (999,000 inputs); quotient truncated, remainder x - (x / y) y.
- Calling:

```
cell  contents   notes
 10     130      CLA 30   y
 11     688      STO 88   into udvm-m's y cell
 12     131      CLA 31   x into the accumulator
 13     865      JMP 65   call udvm-m
 14     632      STO 32   the quotient, from the accumulator
 15     187      CLA 87   the remainder, from cell 87
```

  Quotient in the accumulator, remainder in cell 87.
- Destroys: the accumulator, its cells 65-98 except y in cell 88, and cell 99.
- Cells: 34, 65-98.
- Instructions: worst 1,178, mean 28.6.
- Algorithm: one level of decimal long division: take 10y from x while x / 10 >= y, adding 10 to the quotient,
  then take y while it fits, adding 1. The test x / 10 >= y never forms 10y when it would not fit; 10y is stored
  wrong for y >= 100, where it is never used.
- Sources: [ANDREE]; [IBM1401].
- Status: correct by exhaustive check on HEARTBURN of all 999,000 inputs, called one after another on one load,
  and again after moving it to start at cell 20. Size: not claimed smallest; inline core 27 cells, as called 34;
  any x / y needs at least 8. Speed: exact.
- Moving it: subtract k from the address of every instruction that names a cell in 65-98; 00 and 99 stay, and so
  does the constant 10 in cell 85. Address bases: none.

## udvm-f

- Range: x from 0 to 999, y from 1 to 999 (999,000 inputs); quotient truncated, remainder x - (x / y) y.
- Calling:

```
cell  contents   notes
 10     130      CLA 30   y
 11     679      STO 79   into udvm-f's y cell
 12     131      CLA 31   x into the accumulator
 13     861      JMP 61   call udvm-f
 14     632      STO 32   the quotient, from the accumulator
 15     178      CLA 78   the remainder, from cell 78
```

  Quotient in the accumulator, remainder in cell 78.
- Destroys: the accumulator, its cells 61-98 except y in cell 79, and cell 99.
- Cells: 38, 61-98.
- Instructions: worst 275, mean 30.3.
- Algorithm: decimal long division, the quotient built by Horner's rule (q = 10q at each level, q + 1 at each
  subtraction).
- Sources: [IBM650]; [IBM1401].
- Status: correct by exhaustive check on HEARTBURN of all 999,000 inputs, called one after another on one load,
  and again after moving it to start at cell 20. Size: not claimed smallest; inline core 31 cells, as called 38.
  Speed: exact.
- Moving it: subtract k from the address of every instruction that names a cell in 61-98; 00 and 99 stay.
  Address bases: none.

## urem-s

- Range: x from 0 to 999, y from 1 to 999 (999,000 inputs).
- Calling:

```
cell  contents   notes
 10     130      CLA 30   y
 11     696      STO 96   into urem-s's y cell
 12     131      CLA 31   x into the accumulator
 13     888      JMP 88   call urem-s
 14     632      STO 32   x % y, from the accumulator
```

  Remainder in the accumulator.
- Destroys: the accumulator, its cells 88-98 except y in cell 96, and cell 99.
- Cells: 11, 88-98.
- Instructions: worst 3,005, mean 17.9.
- Algorithm: repeated subtraction with the running remainder in the accumulator, one y added back.
- Sources: [ANDREE].
- Status: correct by exhaustive check on HEARTBURN of all 999,000 inputs, called one after another on one load,
  and again after moving it to start at cell 20. Size: proven smallest in the inline model, 5 cells; no x % y of 4
  cells or fewer exists, with SFT or without (exhaustive search). As called, 11 cells; no bound for the called form.
  Speed: exact.
- Moving it: subtract k from the address of every instruction that names a cell in 88-98; 00 and 99 stay.
  Address bases: none.

## sdvm-s

- Range: x and y from -999 to 999, y not 0 (3,994,002 inputs); quotient truncated toward zero, remainder with the
  sign of x.
- Calling:

```
cell  contents   notes
 10     130      CLA 30   y
 11     689      STO 89   into sdvm-s's y cell
 12     131      CLA 31   x into the accumulator
 13     852      JMP 52   call sdvm-s
 14     632      STO 32   the quotient, from the accumulator
 15     188      CLA 88   the remainder, from cell 88
```

  Quotient in the accumulator, remainder in cell 88.
- Destroys: the accumulator, its cells 52-98, including y in cell 89 (negated when x < 0), and cell 99.
- Cells: 47, 52-98.
- Instructions: worst 8,021, mean 49.8.
- Algorithm: repeated subtraction, counting up for y > 0 and down for y < 0, after x and y are both negated when
  x < 0; the remainder given x's sign at the end.
- Sources: [ANDREE]; [IBM650]; [IBM1620].
- Status: correct by exhaustive check on HEARTBURN of all 3,994,002 inputs, called one after another on one load,
  and again after moving it to start at cell 20. Size: inline model at least 8 cells (no signed x / y or x % y of 7
  or fewer exists, with SFT or without; exhaustive search); core 40, as called 47; the gap from 8 to 40 is open.
  Speed: exact.
- Moving it: subtract k from the address of every instruction that names a cell in 52-98; 00 and 99 stay.
  Address bases: none.

## sdvm-f

- Range: x and y from -999 to 999, y not 0 (3,994,002 inputs); quotient truncated toward zero, remainder with the
  sign of x.
- Calling:

```
cell  contents   notes
 10     130      CLA 30   y
 11     654      STO 54   into sdvm-f's y cell
 12     131      CLA 31   x into the accumulator
 13     828      JMP 28   call sdvm-f
 14     632      STO 32   the quotient, from the accumulator
 15     153      CLA 53   the remainder, from cell 53
```

  Quotient in the accumulator, remainder in cell 53.
- Destroys: the accumulator, its cells 28-98, including y in cell 54 (made |y|), and cell 99.
- Cells: 71, 28-98.
- Instructions: worst 304, mean 50.3.
- Algorithm: decimal long division of the magnitudes, the quotient built by Horner's rule; the signs put back.
- Sources: [IBM650]; [IBM1401]; [IBM1620].
- Status: correct by exhaustive check on HEARTBURN of all 3,994,002 inputs, called one after another on one load,
  and again after moving it to start at cell 20. Size: not claimed smallest; core 65 cells, as called 71.
  Speed: exact.
- Moving it: subtract k from the address of every instruction that names a cell in 28-98; 00 and 99 stay.
  Address bases: none.

## srem-s

- Range: x and y from -999 to 999, y not 0 (3,994,002 inputs); remainder with the sign of x.
- Calling:

```
cell  contents   notes
 10     130      CLA 30   y
 11     692      STO 92   into srem-s's y cell
 12     131      CLA 31   x into the accumulator
 13     875      JMP 75   call srem-s
 14     632      STO 32   x % y, from the accumulator
```

  Remainder in the accumulator.
- Destroys: the accumulator, its cells 75-98, including y in cell 92 (made |y|), and cell 99.
- Cells: 24, 75-98.
- Instructions: worst 3,013, mean 22.7.
- Algorithm: repeated subtraction of |y| for x >= 0; for x < 0, repeated addition with the accumulator held at
  r - 1, so that TAC tests r + y <= 0.
- Sources: [ANDREE]; [IBM650]; [IBM1620].
- Status: correct by exhaustive check on HEARTBURN of all 3,994,002 inputs, called one after another on one load,
  and again after moving it to start at cell 20. Size: inline model at least 8 cells (exhaustive search); core 18,
  as called 24; the gap from 8 to 18 is open. Speed: exact.
- Moving it: subtract k from the address of every instruction that names a cell in 75-98; 00 and 99 stay.
  Address bases: none.

## Sources

Before 1968:

- [ANDREE] Andree, R. V. Programming the IBM 650 Magnetic Drum Computer and Data-Processing Machine. Holt, Rinehart and Winston, 1958. pp. 49-53: "division is accomplished by repeated subtraction, just as it is on a desk calculator".
- [IBM650] IBM. 650 Magnetic Drum Data-Processing Machine, Manual of Operation. Form 22-6060-1, 1955. p. 36: division by repeated subtraction with shifting, "essentially the same as the paper and pencil process"; pp. 36-37: the remainder has the sign of the dividend.
- [IBM1401] IBM. Programs for IBM 1401 Card Systems, Preliminary Specifications. J24-0209-2, 1960. pp. 13-14: the Divide subroutine, by repetitive subtraction with a shift factor aligning divisor and dividend; the remainder has the sign of the dividend.
- [IBM1620] IBM. IBM 1620 Data Processing System Reference Manual. Form A26-4500-2, 1961. p. 21: the remainder has the sign of the dividend.

After 1968:

- [GRANLUND] Granlund, T. and Montgomery, P. L. Division by invariant integers using multiplication. PLDI 1994, SIGPLAN Notices 29(6), 1994.
- [ROBISON] Robison, A. D. N-bit unsigned division via N-bit multiply-add. 17th IEEE Symposium on Computer Arithmetic, 2005.
- [LEMIRE19] Lemire, D., Kaser, O. and Kurz, N. Faster remainder by direct computation: applications to compilers and software libraries. Software: Practice and Experience 49(6), 2019.
- [LEMIRE21] Lemire, D., Bartlett, C. and Kaser, O. Integer division by constants: optimal bounds. Heliyon 7(6), 2021.
