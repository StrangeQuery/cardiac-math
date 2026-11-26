# trig

Sine, cosine, tangent, arctangent and atan2 in whole degrees, for CARDIAC.

| Name | Computes | Format and range | Cells | Instructions worst / mean | Worst error | Destroys |
|---|---|---|---|---|---|---|
| [`sin-s`](#sin-s) | 1000 sin x | x 0..359 degrees; -999..999 | 33-98 (66) | 112 / 70.8 | 3 thousandths | 99, 64-65, 67, 72-73, 97-98 |
| **[`sin-m`](#sin-m)** | 1000 sin x | x 0..359 degrees; -999..999 | 27-98 (72) | 112 / 73.5 | 1 thousandth | 99, 59-60, 62, 66-67, 89, 98 |
| [`sin-f`](#sin-f) | 1000 sin x | x 0..359 degrees; -999..999 | 18-98 (81) | 52 / 45.5 | 1 thousandth | 99, 48-49, 52-53, 58-60, 89, 98 |
| [`scs-s`](#scs-s) | 1000 sin x and 1000 cos x | x 0..359 degrees; -999..999 | 32-98 (67) | sin 112 / 70.8; cos 113 / 72.0 | sin 3; cos 3 thousandths | 99, 64-65, 67, 72-73, 97-98 |
| **[`scs-a`](#scs-a)** | 1000 sin x and 1000 cos x | x 0..359 degrees; -999..999 | 22-98 (77) | sin 112 / 73.5; cos 115 / 76.8 | sin 1; cos 1 thousandths | 99, 58-59, 61, 66-67, 89, 98 |
| [`tan-s`](#tan-s) | 100 tan x | x 0..359 degrees; -999..999, saturating at x mod 180 = 85..95 | 13-98 (86) | 21,113 / 15,902.7 | 1.362 hundredths | 99, 77-85, 97-98 |
| [`atn-s`](#atn-s) | atan(n / 100) | n -999..999 for n / 100; -84..84 degrees | 26-98 (73) | 7,908 / 6,728.0 | 0.694 degrees | 99, 83-90, 98 |
| [`atn-m`](#atn-m) | atan(n / 100) | n -999..999 for n / 100; -84..84 degrees | 22-98 (77) | 8,300 / 6,704.5 | 0.573 degrees | 99, 82-89, 98 |
| [`atn-a`](#atn-a) | atan(n / 100) | n -999..999 for n / 100; -84..84 degrees | 18-98 (81) | 10,109 / 8,168.4 | 0.567 degrees | 99, 81-88, 98 |
| [`at2-s`](#at2-s) | atan2(y, x) | y, x -999..999, not both 0; 0..359 degrees | 12-98 (87) | 7,924 / 7,139.2 | 0.699 degrees | 99, 77-78, 80-85, 98 |

Default: **sin-m**, error 1 (one thousandth) at a modest size, 72 cells and 112 instructions at most. For the sine and the cosine together, **scs-a**: the same table and error with a second entry, in 77 cells.

- Instructions count from the routine's entry to its return jump, both included; the caller's JMP is not counted.
- Errors for tan, atn and at2 are the distance from the exact value (a correctly rounded routine has at most 0.5); for the others, the difference from the correctly rounded value.
- Destroys: cell 99 and the routine's own cells listed; no other cell.

## When to choose another

- `sin-s` when 6 cells matter (66 against 72) and an error of 3 thousandths is fine.
- `sin-f` in inner loops: 52 instructions at most against 112, for 9 more cells (81).
- `scs-s` or `scs-a` when both the sine and the cosine are needed: one table, two entries. `scs-s` is 10 cells smaller at error 3.
- `tan-s` for 100 tan x; it saturates near 90 and 270 degrees.
- `atn-s` is the smallest arctangent (73 cells, 0.694 degrees), `atn-m` lies between (77 cells, 0.573), and `atn-a` is for accuracy (81 cells, 0.567 degrees, 20 inputs misrounded against 86 and 282).
- `at2-s` for a direction from (y, x), 0..359 degrees, the quadrant found from the signs.
- For a function sampled at a few dozen points, a table read (a built CLA: four cells, four or five instructions) beats every routine here, as the notebooks show.

## Routines

### sin-s

1000 sin x, x in whole degrees, from a table every 10 degrees.

- Format and range:
  - Input: x, a whole number of degrees, 0..359.
  - Result: round(1000 sin x), halves away from zero, held to -999..999.
  - The cap: STO keeps three digits and would store 1000 as 000, so 1000 is held to 999. sin 89, 90, 91 give 999 and sin 269, 270, 271 give -999; the reference is held the same way.
- Calling:

  ```
  cell  contents  notes
   03      106    CLA 06  x
   04      833    JMP 33  call sin-s; JMP leaves the return jump in cell 99
   05      607    STO 07  1000 sin x, from the accumulator
   06      030    x = 30 degrees
   07      000    receives 1000 sin x
  ```

  The result is in the accumulator when cell 05 runs: 500.
- Destroys: cell 99 and its own cells 64-65, 67, 72-73, 97-98; the accumulator returns the result.
- Cells: 33-98 (66).
- Instructions: worst 112, mean 70.8.
- Worst error: 3 in the last digit (thousandths) against the correctly rounded value, at x = 60; 196 of 360 inputs off.
- Algorithm: sine table with linear interpolation; the angle mapped to b = 0..180 with sin x = sin(b - 90), so the table covers -90..90 degrees and no sign is restored; 19 entries at a step of 10 degrees, chosen by dynamic programming to minimise the worst interpolated error (as CARDIAC computes it, truncation included) rather than rounded; index and fraction by SFT 01 and SFT 33; table reads are built instructions; D r added in a loop, the sum offset by -700 to stay within a cell.
- Sources:
  - Before 1968: the Aryabhatiya, A.D. 499: 24 sines at steps of 3 3/4 degrees (secondhand, through Gupta 1967); R. C. Gupta, "Bhaskara I's approximation to sine", Indian Journal of History of Science 2(2), 121-136, 1967 (read).
  - Library sine routines of the day (polynomials, not tables): IBM, Additional Abstracts of IBM 650 Programs, March 1957, abstracts 3.1.011 and 3.1.021 (read); Royal McBee, LGP-30 Subroutine Manual, October 1960, Program 14.0, sine-cosine subroutine in degrees (read).
  - Choosing table entries by dynamic programming for the interpolation: no earlier source known; none was searched for.
- Status: worst error and misrounded count exact, by exhaustive check on HEARTBURN over the whole domain (360 inputs), called at these addresses through the convention, from one load, in two orders, and again moved down. No size is claimed minimal. The table's entries are optimal for this interpolation among entries within 4 units of the rounded sines (exact dynamic program over the entries).
- Moving it: subtract d from every address that points into cells 33-98 (00 and 99 stay), address bases included: cell 71 (ADD 75).

### sin-m

1000 sin x, x in whole degrees, from a table every 5 degrees.

- Format and range:
  - Input: x, a whole number of degrees, 0..359.
  - Result: round(1000 sin x), halves away from zero, held to -999..999.
  - The cap: STO keeps three digits and would store 1000 as 000, so 1000 is held to 999. sin 89, 90, 91 give 999 and sin 269, 270, 271 give -999; the reference is held the same way.
- Calling:

  ```
  cell  contents  notes
   03      106    CLA 06  x
   04      827    JMP 27  call sin-m; JMP leaves the return jump in cell 99
   05      607    STO 07  1000 sin x, from the accumulator
   06      030    x = 30 degrees
   07      000    receives 1000 sin x
  ```

  The result is in the accumulator when cell 05 runs: 500.
- Destroys: cell 99 and its own cells 59-60, 62, 66-67, 89, 98; the accumulator returns the result.
- Cells: 27-98 (72).
- Instructions: worst 112, mean 73.5.
- Worst error: 1 in the last digit (thousandths) against the correctly rounded value, at x = 1; 88 of 360 inputs off.
- Algorithm: sine table with quarter-wave symmetry and linear interpolation; 19 entries of cos 5k over 0..90 degrees, chosen by dynamic programming to minimise the worst interpolated error rather than rounded; the doubled angle gives index and fraction by SFT 01 and SFT 33; table reads are built instructions; D r added in a loop.
- Sources:
  - Before 1968: the Aryabhatiya, A.D. 499: 24 sines at steps of 3 3/4 degrees (secondhand, through Gupta 1967); R. C. Gupta, "Bhaskara I's approximation to sine", Indian Journal of History of Science 2(2), 121-136, 1967 (read).
  - Library sine routines of the day (polynomials, not tables): IBM, Additional Abstracts of IBM 650 Programs, March 1957, abstracts 3.1.011 and 3.1.021 (read); Royal McBee, LGP-30 Subroutine Manual, October 1960, Program 14.0, sine-cosine subroutine in degrees (read).
  - Choosing table entries by dynamic programming for the interpolation: no earlier source known; none was searched for.
- Status: worst error and misrounded count exact, by exhaustive check on HEARTBURN over the whole domain (360 inputs), called at these addresses through the convention, from one load, in two orders, and again moved down. No size is claimed minimal. The table's entries are optimal for this interpolation among entries within 4 units of the rounded sines (exact dynamic program over the entries).
- Moving it: subtract d from every address that points into cells 27-98 (00 and 99 stay), address bases included: cell 65 (ADD 69).

### sin-f

1000 sin x, x in whole degrees, from a table every 5 degrees, interpolated without a loop.

- Format and range:
  - Input: x, a whole number of degrees, 0..359.
  - Result: round(1000 sin x), halves away from zero, held to -999..999.
  - The cap: STO keeps three digits and would store 1000 as 000, so 1000 is held to 999. sin 89, 90, 91 give 999 and sin 269, 270, 271 give -999; the reference is held the same way.
- Calling:

  ```
  cell  contents  notes
   03      106    CLA 06  x
   04      818    JMP 18  call sin-f; JMP leaves the return jump in cell 99
   05      607    STO 07  1000 sin x, from the accumulator
   06      030    x = 30 degrees
   07      000    receives 1000 sin x
  ```

  The result is in the accumulator when cell 05 runs: 500.
- Destroys: cell 99 and its own cells 48-49, 52-53, 58-60, 89, 98; the accumulator returns the result.
- Cells: 18-98 (81).
- Instructions: worst 52, mean 45.5.
- Worst error: 1 in the last digit (thousandths) against the correctly rounded value, at x = 1; 88 of 360 inputs off.
- Algorithm: sine table with quarter-wave symmetry and linear interpolation, as sin-m (the same 19 entries chosen by dynamic programming); D r added by a computed jump into a chain of eight ADD D, the sum kept in the accumulator.
- Sources:
  - Before 1968: the Aryabhatiya, A.D. 499: 24 sines at steps of 3 3/4 degrees (secondhand, through Gupta 1967); R. C. Gupta, "Bhaskara I's approximation to sine", Indian Journal of History of Science 2(2), 121-136, 1967 (read).
  - Library sine routines of the day (polynomials, not tables): IBM, Additional Abstracts of IBM 650 Programs, March 1957, abstracts 3.1.011 and 3.1.021 (read); Royal McBee, LGP-30 Subroutine Manual, October 1960, Program 14.0, sine-cosine subroutine in degrees (read).
  - Choosing table entries by dynamic programming for the interpolation: no earlier source known; none was searched for.
- Status: worst error and misrounded count exact, by exhaustive check on HEARTBURN over the whole domain (360 inputs), called at these addresses through the convention, from one load, in two orders, and again moved down. No size is claimed minimal. The table's entries are optimal for this interpolation among entries within 4 units of the rounded sines (exact dynamic program over the entries).
- Moving it: subtract d from every address that points into cells 18-98 (00 and 99 stay), address bases included: cell 56 (ADD 62), cell 57 (JMP 88).

### scs-s

1000 sin x and 1000 cos x, x in whole degrees, from one table every 10 degrees, with an entry for each.

- Format and range:
  - Input: x, a whole number of degrees, 0..359.
  - Result: round(1000 sin x) or round(1000 cos x), halves away from zero, held to -999..999.
  - The cap: STO keeps three digits and would store 1000 as 000, so 1000 is held to 999. sin 89-91 and cos 359, 0, 1 give 999; sin 269-271 and cos 179-181 give -999; the reference is held the same way.
- Calling:

  ```
  cell  contents  notes
   03      109    CLA 09  x
   04      833    JMP 33  the sine's entry; JMP leaves the return jump in cell 99
   05      610    STO 10  1000 sin x, from the accumulator
   06      109    CLA 09  x again
   07      832    JMP 32  the cosine's entry
   08      611    STO 11  1000 cos x, from the accumulator
   09      030    x = 30 degrees
   10      000    receives 1000 sin x
   11      000    receives 1000 cos x
  ```

  The sine is in the accumulator when cell 05 runs (500 for x = 30), the cosine when cell 08 runs (869).
- Destroys: cell 99 and its own cells 64-65, 67, 72-73, 97-98; the accumulator returns the result.
- Cells: 32-98 (67).
- Instructions, sin: worst 112, mean 70.8.
- Instructions, cos: worst 113, mean 72.0.
- Worst error, sin: 3 in the last digit (thousandths) against the correctly rounded value, at x = 60; 196 of 360 inputs off.
- Worst error, cos: 3 in the last digit (thousandths) against the correctly rounded value, at x = 3; 196 of 360 inputs off.
- Algorithm: sine table with linear interpolation; the angle mapped to b = 0..180 with sin x = sin(b - 90), so the table covers -90..90 degrees and no sign is restored; 19 entries at a step of 10 degrees, chosen by dynamic programming to minimise the worst interpolated error (as CARDIAC computes it, truncation included) rather than rounded; index and fraction by SFT 01 and SFT 33; table reads are built instructions; D r added in a loop, the sum offset by -700 to stay within a cell; the cosine as sin(x + 90): one ADD in front of the sine's entry.
- Sources:
  - Before 1968: the Aryabhatiya, A.D. 499: 24 sines at steps of 3 3/4 degrees (secondhand, through Gupta 1967); R. C. Gupta, "Bhaskara I's approximation to sine", Indian Journal of History of Science 2(2), 121-136, 1967 (read).
  - Library sine routines of the day (polynomials, not tables): IBM, Additional Abstracts of IBM 650 Programs, March 1957, abstracts 3.1.011 and 3.1.021 (read); Royal McBee, LGP-30 Subroutine Manual, October 1960, Program 14.0, sine-cosine subroutine in degrees (read).
  - Choosing table entries by dynamic programming for the interpolation: no earlier source known; none was searched for.
- Status: worst error and misrounded count exact, by exhaustive check on HEARTBURN over the whole domain (360 inputs, each entry), called at these addresses through the convention, from one load, in two orders, and again moved down. No size is claimed minimal. The table's entries are optimal for this interpolation among entries within 4 units of the rounded sines (exact dynamic program over the entries).
- Moving it: subtract d from every address that points into cells 32-98 (00 and 99 stay), address bases included: cell 71 (ADD 75).

### scs-a

1000 sin x and 1000 cos x, x in whole degrees, from one table every 5 degrees, with an entry for each.

- Format and range:
  - Input: x, a whole number of degrees, 0..359.
  - Result: round(1000 sin x) or round(1000 cos x), halves away from zero, held to -999..999.
  - The cap: STO keeps three digits and would store 1000 as 000, so 1000 is held to 999. sin 89-91 and cos 359, 0, 1 give 999; sin 269-271 and cos 179-181 give -999; the reference is held the same way.
- Calling:

  ```
  cell  contents  notes
   03      109    CLA 09  x
   04      826    JMP 26  the sine's entry; JMP leaves the return jump in cell 99
   05      610    STO 10  1000 sin x, from the accumulator
   06      109    CLA 09  x again
   07      822    JMP 22  the cosine's entry
   08      611    STO 11  1000 cos x, from the accumulator
   09      030    x = 30 degrees
   10      000    receives 1000 sin x
   11      000    receives 1000 cos x
  ```

  The sine is in the accumulator when cell 05 runs (500 for x = 30), the cosine when cell 08 runs (866).
- Destroys: cell 99 and its own cells 58-59, 61, 66-67, 89, 98; the accumulator returns the result.
- Cells: 22-98 (77).
- Instructions, sin: worst 112, mean 73.5.
- Instructions, cos: worst 115, mean 76.8.
- Worst error, sin: 1 in the last digit (thousandths) against the correctly rounded value, at x = 1; 88 of 360 inputs off.
- Worst error, cos: 1 in the last digit (thousandths) against the correctly rounded value, at x = 3; 88 of 360 inputs off.
- Algorithm: sine table with quarter-wave symmetry and linear interpolation; 19 entries of cos 5k over 0..90 degrees, chosen by dynamic programming to minimise the worst interpolated error rather than rounded; the doubled angle gives index and fraction by SFT 01 and SFT 33; table reads are built instructions; D r added in a loop; the cosine as sin(x + 90) modulo 360: four cells in front of the sine's entry, reaching it without a JMP (a JMP would overwrite the return in cell 99).
- Sources:
  - Before 1968: the Aryabhatiya, A.D. 499: 24 sines at steps of 3 3/4 degrees (secondhand, through Gupta 1967); R. C. Gupta, "Bhaskara I's approximation to sine", Indian Journal of History of Science 2(2), 121-136, 1967 (read).
  - Library sine routines of the day (polynomials, not tables): IBM, Additional Abstracts of IBM 650 Programs, March 1957, abstracts 3.1.011 and 3.1.021 (read); Royal McBee, LGP-30 Subroutine Manual, October 1960, Program 14.0, sine-cosine subroutine in degrees (read).
  - Choosing table entries by dynamic programming for the interpolation: no earlier source known; none was searched for.
- Status: worst error and misrounded count exact, by exhaustive check on HEARTBURN over the whole domain (360 inputs, each entry), called at these addresses through the convention, from one load, in two orders, and again moved down. No size is claimed minimal. The table's entries are optimal for this interpolation among entries within 4 units of the rounded sines (exact dynamic program over the entries).
- Moving it: subtract d from every address that points into cells 22-98 (00 and 99 stay), address bases included: cell 65 (ADD 69).

### tan-s

100 tan x, x in whole degrees, by partial fractions.

- Format and range:
  - Input: x, a whole number of degrees, 0..359.
  - Result: round(100 tan x) where it fits a cell, x mod 180 = 0..84 and 96..179 (-951..951).
  - Saturation: 999 for x mod 180 = 85..90, -999 for 91..95.
- Calling:

  ```
  cell  contents  notes
   03      106    CLA 06  x
   04      813    JMP 13  call tan-s; JMP leaves the return jump in cell 99
   05      607    STO 07  100 tan x, from the accumulator
   06      045    x = 45 degrees
   07      000    receives 100 tan x
  ```

  The result is in the accumulator when cell 05 runs: 100.
- Destroys: cell 99 and its own cells 77-85, 97-98; the accumulator returns the result.
- Cells: 13-98 (86).
- Instructions: worst 21,113, mean 15,902.7.
- Worst error: 1.362 hundredths from the exact value, at x = 54; 124 of 360 inputs not correctly rounded.
- Algorithm: partial-fraction tangent (Euler): 100 tan d = A / (90 - d) - A / (90 + d) + R(d), A = 18000 / pi; computed as floor(5730 / (90 - d)) - floor(5730 / (90 + d)) + floor(6d / 17), constants by exhaustive search over the forms the multiply computes; each term an exact floor(m a / b) by a Bresenham-style error-accumulation loop that stores nothing larger than b; d mod 180 folded to 0..90, the sign put back by a stored ADD or SUB.
- Sources:
  - Before 1968: L. Euler, the partial-fraction expansion of the tangent, eighteenth century (secondhand); J. E. Bresenham, "Algorithm for computer control of a digital plotter", IBM Systems Journal 4(1), 1965 (secondhand: the loop is the standard one).
- Status: worst error and misrounded count exact, by exhaustive check on HEARTBURN over the whole domain (360 inputs), called at these addresses through the convention, from one load, in two orders, and again moved down. No size is claimed minimal. Proven impossible here: correct rounding everywhere. 100 tan d is within 0.008 of a half at d = 2, 18 and 68, so no formula in three- and four-digit arithmetic rounds it correctly at every d, and an exact table (41 cells) does not fit beside its code.
- Moving it: subtract d from every address that points into cells 13-98 (00 and 99 stay), address bases included: cell 89 (ADD 85), cell 90 (SUB 85).

### atn-s

atan(n / 100), in whole degrees, by a polynomial of two terms.

- Format and range:
  - Input: n, -999..999, standing for the ratio n / 100.
  - Result: atan(n / 100) in whole degrees, rounded, -84..84.
- Calling:

  ```
  cell  contents  notes
   03      106    CLA 06  n
   04      826    JMP 26  call atn-s; JMP leaves the return jump in cell 99
   05      607    STO 07  atan(n / 100) in degrees, from the accumulator
   06     -173    n = -173, the ratio -1.73
   07      000    receives atan(n / 100) in degrees
  ```

  The result is in the accumulator when cell 05 runs: -60.
- Destroys: cell 99 and its own cells 83-90, 98; the accumulator returns the result.
- Cells: 26-98 (73).
- Instructions: worst 7,908, mean 6,728.0.
- Worst error: 0.694 degrees from the exact value, at n = -780; 282 of 1,999 inputs not correctly rounded.
- Algorithm: Horner polynomial arctangent with octant folding: z = min(|n|, 100) / max(|n|, 100), t = atan z in tenths of a degree as a polynomial in z, each product an exact floor(m a / b) by a Bresenham-style loop; coefficients are minimax fits tuned by coordinate descent on the integer pipeline; then 90 - t for |n| > 100 and the sign; two terms, z (604 - 152 z).
- Sources:
  - Before 1968: C. Hastings, Jr., Approximations for Digital Computers, Princeton University Press, 1955 (not obtained; its arctangent forms read as reprinted in Abramowitz and Stegun); M. Abramowitz and I. A. Stegun (eds.), Handbook of Mathematical Functions, National Bureau of Standards, Applied Mathematics Series 55, 1964 (read in part), 4.4.47-4.4.48, p. 81; J. E. Bresenham, "Algorithm for computer control of a digital plotter", IBM Systems Journal 4(1), 1965 (secondhand: the loop is the standard one).
  - The coefficients are this library's own fits, not Hastings's.
- Status: worst error and misrounded count exact, by exhaustive check on HEARTBURN over the whole domain (1,999 inputs), called at these addresses through the convention, from one load, in two orders, and again moved down. No size is claimed minimal. Proven impossible here: a correctly rounded atan(n / 100) needs a table of its 84 thresholds (41 cells packed) plus about 25 cells to walk it, more than the machine leaves.
- Moving it: subtract d from every address that points into cells 26-98 (00 and 99 stay); it holds no address bases.

### atn-m

atan(n / 100), in whole degrees, by a polynomial of three terms.

- Format and range:
  - Input: n, -999..999, standing for the ratio n / 100.
  - Result: atan(n / 100) in whole degrees, rounded, -84..84.
- Calling:

  ```
  cell  contents  notes
   03      106    CLA 06  n
   04      822    JMP 22  call atn-m; JMP leaves the return jump in cell 99
   05      607    STO 07  atan(n / 100) in degrees, from the accumulator
   06     -173    n = -173, the ratio -1.73
   07      000    receives atan(n / 100) in degrees
  ```

  The result is in the accumulator when cell 05 runs: -60.
- Destroys: cell 99 and its own cells 82-89, 98; the accumulator returns the result.
- Cells: 22-98 (77).
- Instructions: worst 8,300, mean 6,704.5.
- Worst error: 0.573 degrees from the exact value, at n = -675; 86 of 1,999 inputs not correctly rounded.
- Algorithm: Horner polynomial arctangent with octant folding: z = min(|n|, 100) / max(|n|, 100), t = atan z in tenths of a degree as a polynomial in z, each product an exact floor(m a / b) by a Bresenham-style loop; coefficients are minimax fits tuned by coordinate descent on the integer pipeline; then 90 - t for |n| > 100 and the sign; three terms, z (588 - z (95 + 45 z)).
- Sources:
  - Before 1968: C. Hastings, Jr., Approximations for Digital Computers, Princeton University Press, 1955 (not obtained; its arctangent forms read as reprinted in Abramowitz and Stegun); M. Abramowitz and I. A. Stegun (eds.), Handbook of Mathematical Functions, National Bureau of Standards, Applied Mathematics Series 55, 1964 (read in part), 4.4.47-4.4.48, p. 81; J. E. Bresenham, "Algorithm for computer control of a digital plotter", IBM Systems Journal 4(1), 1965 (secondhand: the loop is the standard one).
  - The coefficients are this library's own fits, not Hastings's.
- Status: worst error and misrounded count exact, by exhaustive check on HEARTBURN over the whole domain (1,999 inputs), called at these addresses through the convention, from one load, in two orders, and again moved down. No size is claimed minimal. Proven impossible here: a correctly rounded atan(n / 100) needs a table of its 84 thresholds (41 cells packed) plus about 25 cells to walk it, more than the machine leaves.
- Moving it: subtract d from every address that points into cells 22-98 (00 and 99 stay); it holds no address bases.

### atn-a

atan(n / 100), in whole degrees, by a polynomial of four terms.

- Format and range:
  - Input: n, -999..999, standing for the ratio n / 100.
  - Result: atan(n / 100) in whole degrees, rounded, -84..84.
- Calling:

  ```
  cell  contents  notes
   03      106    CLA 06  n
   04      818    JMP 18  call atn-a; JMP leaves the return jump in cell 99
   05      607    STO 07  atan(n / 100) in degrees, from the accumulator
   06     -173    n = -173, the ratio -1.73
   07      000    receives atan(n / 100) in degrees
  ```

  The result is in the accumulator when cell 05 runs: -60.
- Destroys: cell 99 and its own cells 81-88, 98; the accumulator returns the result.
- Cells: 18-98 (81).
- Instructions: worst 10,109, mean 8,168.4.
- Worst error: 0.567 degrees from the exact value, at n = -102; 20 of 1,999 inputs not correctly rounded.
- Algorithm: Horner polynomial arctangent with octant folding: z = min(|n|, 100) / max(|n|, 100), t = atan z in tenths of a degree as a polynomial in z, each product an exact floor(m a / b) by a Bresenham-style loop; coefficients are minimax fits tuned by coordinate descent on the integer pipeline; then 90 - t for |n| > 100 and the sign; four terms, z (574 - z (8 + z (197 - 81 z))).
- Sources:
  - Before 1968: C. Hastings, Jr., Approximations for Digital Computers, Princeton University Press, 1955 (not obtained; its arctangent forms read as reprinted in Abramowitz and Stegun); M. Abramowitz and I. A. Stegun (eds.), Handbook of Mathematical Functions, National Bureau of Standards, Applied Mathematics Series 55, 1964 (read in part), 4.4.47-4.4.48, p. 81; J. E. Bresenham, "Algorithm for computer control of a digital plotter", IBM Systems Journal 4(1), 1965 (secondhand: the loop is the standard one).
  - The coefficients are this library's own fits, not Hastings's.
- Status: worst error and misrounded count exact, by exhaustive check on HEARTBURN over the whole domain (1,999 inputs), called at these addresses through the convention, from one load, in two orders, and again moved down. No size is claimed minimal. Proven impossible here: a correctly rounded atan(n / 100) needs a table of its 84 thresholds (41 cells packed) plus about 25 cells to walk it, more than the machine leaves.
- Moving it: subtract d from every address that points into cells 18-98 (00 and 99 stay); it holds no address bases.

### at2-s

atan2(y, x), the direction of (x, y) in whole degrees.

- Format and range:
  - Input: y and x, -999..999, not both 0.
  - Result: the direction of (x, y) anticlockwise from the positive x axis, whole degrees, rounded, 0..359 (360 is 0).
- Calling:

  ```
  cell  contents  notes
   03      109    CLA 09  x
   04      679    STO 79  into at2-s's operand cell for x
   05      108    CLA 08  y
   06      812    JMP 12  call at2-s; JMP leaves the return jump in cell 99
   07      610    STO 10  the direction, from the accumulator
   08     -500    y = -500
   09     -866    x = -866
   10      000    receives the direction in degrees
  ```

  The result is in the accumulator when cell 07 runs: 210.
- Destroys: cell 99 and its own cells 77-78, 80-85, 98; the accumulator returns the result. x's operand cell 79 is kept, so a run of calls with one x stores it once.
- Cells: 12-98 (87).
- Instructions: worst 7,924, mean 7,139.2.
- Worst error: 0.699 degrees from the exact value, at y = -912, x = -890; 462,024 of 3,996,000 inputs not correctly rounded.
- Algorithm: Horner polynomial arctangent with octant folding: a = min(|x|, |y|), b = max, t = atan(a / b) = z (604 - 152 z) tenths of a degree, each product an exact floor(m a / b) by a Bresenham-style loop; then the reflections 90 - t, -t, 180 - t and + 360.
- Sources:
  - Before 1968: C. Hastings, Jr., Approximations for Digital Computers, Princeton University Press, 1955 (not obtained; its arctangent forms read as reprinted in Abramowitz and Stegun); M. Abramowitz and I. A. Stegun (eds.), Handbook of Mathematical Functions, National Bureau of Standards, Applied Mathematics Series 55, 1964 (read in part), 4.4.47-4.4.48, p. 81; J. E. Bresenham, "Algorithm for computer control of a digital plotter", IBM Systems Journal 4(1), 1965 (secondhand: the loop is the standard one).
  - The coefficients are this library's own fits, not Hastings's.
- Status: worst error and misrounded count exact, by exhaustive check on HEARTBURN over the whole domain (3,996,000 inputs), called at these addresses through the convention, from one load, in two orders, and again moved down. No size is claimed minimal. Proven impossible here: correct rounding. Over 0 <= p <= q <= 999 the ratio p / q comes within 8.9 x 10^-8 of a rounding boundary tan(j - 0.5 degrees) (170 / 613 and 15.5 degrees), so a correctly rounded atan2 needs the ratio to about eight significant digits.
- Moving it: subtract d from every address that points into cells 12-98 (00 and 99 stay); it holds no address bases.

## Not shipped

- The fast sine on the 10-degree table (72 cells, error 3): sin-m has error 1 in the same 72 cells, and sin-f is faster.
- Separate cosine routines: the two-entry scs routines give the cosine for 1 cell (scs-s) or 5 cells (scs-a) more than the sine alone.
- The more accurate tangent (error 1.05 hundredths, with floor(5731 / (90 + d)) and floor(27d / 77)): it needs 93 cells as a subroutine and does not fit.
- The three-term atan2 (worst 0.616 degrees): it needs 91 cells as a subroutine and does not fit.
- Polynomial sines of degree 4 and 6, the method of the library sines of the day: error 2 and 1 in 78 and 84 cells (inline, without the call frame) at about 250 and 310 instructions; the tables beat them on every count.
- CORDIC, the method of the HP-35 (Volder 1959; decimal by Meggitt 1962): it needs a guard digit that three-digit cells lack, and the tables beat it. A decimal CORDIC sine with one-cell x and y is off by 15 units; with a four-digit x or y it is 2 units in 86 to 93 cells and 1,160 to 1,660 instructions, against sin-m's 1 unit in 72 cells and 112 instructions. A decimal CORDIC arctangent in three-digit arithmetic is off by 6.9 degrees, the HP-35 tangent by 59 hundredths.

## Sources

Before 1968, so available to a CARDIAC programmer of the day:

- The Aryabhatiya, A.D. 499: 24 sines at steps of 3 3/4 degrees (secondhand, through Gupta 1967).
- L. Euler, the partial-fraction expansion of the tangent, eighteenth century (secondhand).
- C. Hastings, Jr., Approximations for Digital Computers, Princeton University Press, 1955 (not obtained; its arctangent forms read as reprinted in Abramowitz and Stegun).
- IBM, Additional Abstracts of IBM 650 Programs, March 1957, abstracts 3.1.011 and 3.1.021 (read).
- J. E. Volder, "The CORDIC Trigonometric Computing Technique", IRE Transactions on Electronic Computers EC-8(3), 330-334, 1959 (read).
- Royal McBee, LGP-30 Subroutine Manual, October 1960, Program 14.0, sine-cosine subroutine in degrees (read).
- J. E. Meggitt, "Pseudo Division and Pseudo Multiplication Processes", IBM Journal of Research and Development 6(2), 210-226, 1962 (read in full).
- M. Abramowitz and I. A. Stegun (eds.), Handbook of Mathematical Functions, National Bureau of Standards, Applied Mathematics Series 55, 1964 (read in part).
- J. E. Bresenham, "Algorithm for computer control of a digital plotter", IBM Systems Journal 4(1), 1965 (secondhand: the loop is the standard one).
- R. C. Gupta, "Bhaskara I's approximation to sine", Indian Journal of History of Science 2(2), 121-136, 1967 (read).
- D. S. Cochran, "Internal programming of the 9100A calculator", Hewlett-Packard Journal, September 1968, 14-16 (read).
- D. W. Hagelbarger and S. Fingerman, CARDIAC: A Cardboard Illustrative Aid to Computation, Bell Telephone Laboratories, 1968.

After 1968:

- J. S. Walther, "A unified algorithm for elementary functions", AFIPS Spring Joint Computer Conference, 1971, 379-385 (read).
- D. S. Cochran, "Algorithms and Accuracy in the HP-35", Hewlett-Packard Journal, June 1972, 10-11 (read).
