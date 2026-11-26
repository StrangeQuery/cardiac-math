# log

Common and natural logarithms, antilogarithms and e^x, for CARDIAC.

| Name | Computes | Format and range | Cells | Instructions worst / mean | Worst error | Destroys |
|---|---|---|---|---|---|---|
| **[`lgt-s`](#lgt-s)** | 1000 log10 x | x 1..999; 0..3000 (characteristic and mantissa) | 22-98 (77) | 264 / 138.4 | 1 thousandth | 99, 69-71, 98 |
| [`lgt-a`](#lgt-a) | 1000 log10 x | x 1..999; 0..3000 (characteristic and mantissa) | 15-98 (84) | 287 / 152.5 | 1 thousandth | 99, 67-70, 98 |
| [`lnx-s`](#lnx-s) | 1000 ln x | x 1..999; 0..6908 | 19-98 (80) | 252 / 119.9 | 2 thousandths | 99, 67-69, 98 |
| [`lnx-a`](#lnx-a) | 1000 ln x | x 1..999; 0..6908 | 12-98 (87) | 283 / 133.9 | 1 thousandth | 99, 65-68, 98 |
| [`alg-s`](#alg-s) | 100 x 10^(m / 1000) | mantissa 0..999; 100..998 | 12-98 (87) | 911 / 525.3 | 2 in the third digit | 99, 33, 41, 52, 61-66, 98 |
| [`alg-a`](#alg-a) | 100 x 10^(m / 1000) | mantissa 0..999; 100..998 | 09-98 (90) | 943 / 541.0 | 1 in the third digit | 99, 31, 34, 42, 53, 66-71, 98 |
| [`exp-s`](#exp-s) | 100 e^(v / 1000) | v 0..2302 (x = v / 1000); 100..999 | 12-98 (87) | 910 / 524.8 | 1 in the third digit | 99, 33, 41, 52, 62-67, 98 |

Default: **lgt-s**, the common logarithm to within 1 thousandth in the fewest cells (77), laid out as a table of logarithms gives it, characteristic and mantissa.

- Instructions count from the routine's entry to its return jump, both included; the caller's JMP is not counted.
- Errors for tan, atn and at2 are the distance from the exact value (a correctly rounded routine has at most 0.5); for the others, the difference from the correctly rounded value.
- Destroys: cell 99 and the routine's own cells listed; no other cell.

## When to choose another

- `lgt-a` when fewer misrounded results (85% correctly rounded against 76%) and a bound of 1 proven by argument matter, for 7 more cells.
- `lnx-s` or `lnx-a` for natural logarithms: `lnx-s` is 7 cells smaller with error 2, `lnx-a` has error 1.
- `alg-s` or `alg-a` for the antilogarithm of a mantissa: `alg-a` has error 1 for 3 more cells.
- `exp-s` for e^x with x below ln 10; a larger x is reduced as its section says.
- For a function sampled at a few dozen points, a table read (a built CLA: four cells, four or five instructions) beats every routine here, as the notebooks show.

## Routines

### lgt-s

1000 log10 x, for x from 1 to 999, by pseudo-division.

- Format and range:
  - Input: x, 1..999.
  - Result: round(1000 log10 x), 0..3000, as tables of logarithms lay it out: 1000 x characteristic + mantissa. STO keeps the mantissa; SFT 03 leaves the characteristic. log10 200 gives 2301.
  - For x = 999 the correctly rounded result is 3000: characteristic 3, mantissa 000.
- Calling:

  ```
  cell  contents  notes
   03      108    CLA 08  x
   04      822    JMP 22  call lgt-s; JMP leaves the return jump in cell 99
   05      609    STO 09  the last three digits, the mantissa
   06      403    SFT 03  the thousands, the characteristic
   07      610    STO 10
   08      200    x = 200
   09      000    receives the last three digits
   10      000    receives the thousands
  ```

  The result is in the accumulator when cell 05 runs: 2301, more than a cell holds; STO keeps 301 and SFT 03 leaves 2.
- Destroys: cell 99 and its own cells 69-71, 98; the accumulator returns the result.
- Cells: 22-98 (77).
- Instructions: worst 264, mean 138.4.
- Worst error: 1 in the last digit (thousandths) against the correctly rounded value, at x = 9; 241 of 999 inputs off.
- Algorithm: Meggitt's decimal pseudo-division in the form of Briggs's radix method: x doubled to 500..999 (exact), then multiplied by 1.1 and by 1.01 while it stays at most 1000, each factor's logarithm added to a sum held in two cells; the work is on the remainder 1000 - y (Meggitt's renormalised pseudo remainder, so three digits carry five places); what is left is its own logarithm up to a factor (Briggs's and Meggitt's tail rule); three-digit remainder, the 1.1 steps rounded.
- Sources:
  - Before 1968: H. Briggs, Arithmetica Logarithmica, London, 1624, chapter 14 (read in Ian Bruce's translation, and in D. Roegel's reconstruction, LOCOMAT, 2011); J. E. Meggitt, "Pseudo Division and Pseudo Multiplication Processes", IBM Journal of Research and Development 6(2), 210-226, 1962 (read in full), equations 7, 9 and 19; S. Newcomb, Logarithmic and Other Mathematical Tables, Holt, 1882, pp. 3, 9, 22 (read) (characteristic and mantissa).
  - After 1968, the same method in a calculator: D. S. Cochran, "Algorithms and Accuracy in the HP-35", Hewlett-Packard Journal, June 1972, 10-11 (read; after 1968).
- Status: worst error and misrounded count exact, by exhaustive check on HEARTBURN over the whole domain (999 inputs), called at these addresses through the convention, from one load, in two orders, and again moved down. No size is claimed minimal. The bound of one unit is a measurement only: the error before the final rounding reaches 1.06 units. No routine of this kind is correctly rounded everywhere: 1000 log10 x comes within 0.00086 of a half, so correct rounding needs six or seven digits before rounding, against the accumulator's four.
- Moving it: subtract d from every address that points into cells 22-98 (00 and 99 stay); it holds no address bases.

### lgt-a

1000 log10 x, for x from 1 to 999, by pseudo-division with a four-digit remainder.

- Format and range:
  - Input: x, 1..999.
  - Result: round(1000 log10 x), 0..3000, as tables of logarithms lay it out: 1000 x characteristic + mantissa. STO keeps the mantissa; SFT 03 leaves the characteristic. log10 200 gives 2301.
  - For x = 999 the correctly rounded result is 3000: characteristic 3, mantissa 000.
- Calling:

  ```
  cell  contents  notes
   03      108    CLA 08  x
   04      815    JMP 15  call lgt-a; JMP leaves the return jump in cell 99
   05      609    STO 09  the last three digits, the mantissa
   06      403    SFT 03  the thousands, the characteristic
   07      610    STO 10
   08      200    x = 200
   09      000    receives the last three digits
   10      000    receives the thousands
  ```

  The result is in the accumulator when cell 05 runs: 2301, more than a cell holds; STO keeps 301 and SFT 03 leaves 2.
- Destroys: cell 99 and its own cells 67-70, 98; the accumulator returns the result.
- Cells: 15-98 (84).
- Instructions: worst 287, mean 152.5.
- Worst error: 1 in the last digit (thousandths) against the correctly rounded value, at x = 11; 145 of 999 inputs off.
- Algorithm: Meggitt's decimal pseudo-division in the form of Briggs's radix method: x doubled to 500..999 (exact), then multiplied by 1.1 and by 1.01 while it stays at most 1000, each factor's logarithm added to a sum held in two cells; the work is on the remainder 1000 - y (Meggitt's renormalised pseudo remainder, so three digits carry five places); what is left is its own logarithm up to a factor (Briggs's and Meggitt's tail rule); four-digit remainder in the 1.1 steps (its tenths in a cell of their own), which divides their rounding error by ten.
- Sources:
  - Before 1968: H. Briggs, Arithmetica Logarithmica, London, 1624, chapter 14 (read in Ian Bruce's translation, and in D. Roegel's reconstruction, LOCOMAT, 2011); J. E. Meggitt, "Pseudo Division and Pseudo Multiplication Processes", IBM Journal of Research and Development 6(2), 210-226, 1962 (read in full), equations 7, 9 and 19; S. Newcomb, Logarithmic and Other Mathematical Tables, Holt, 1882, pp. 3, 9, 22 (read) (characteristic and mantissa).
  - After 1968, the same method in a calculator: D. S. Cochran, "Algorithms and Accuracy in the HP-35", Hewlett-Packard Journal, June 1972, 10-11 (read; after 1968).
- Status: worst error and misrounded count exact, by exhaustive check on HEARTBURN over the whole domain (999 inputs), called at these addresses through the convention, from one load, in two orders, and again moved down. No size is claimed minimal. The bound of one unit is also proven by argument: the integer model, which agrees with HEARTBURN on every input, has an error before the final rounding of at most 0.64 unit, and rounding is monotone, so the final error is at most 1. No routine of this kind is correctly rounded everywhere: 1000 log10 x comes within 0.00086 of a half, so correct rounding needs six or seven digits before rounding, against the accumulator's four.
- Moving it: subtract d from every address that points into cells 15-98 (00 and 99 stay); it holds no address bases.

### lnx-s

1000 ln x, for x from 1 to 999, by pseudo-division.

- Format and range:
  - Input: x, 1..999.
  - Result: round(1000 ln x), 0..6908, in the accumulator only (STO keeps the last three digits, SFT 03 leaves the thousands).
- Calling:

  ```
  cell  contents  notes
   03      108    CLA 08  x
   04      819    JMP 19  call lnx-s; JMP leaves the return jump in cell 99
   05      609    STO 09  the last three digits
   06      403    SFT 03  the thousands
   07      610    STO 10
   08      100    x = 100
   09      000    receives the last three digits
   10      000    receives the thousands
  ```

  The result is in the accumulator when cell 05 runs: 4605, more than a cell holds; STO keeps 605 and SFT 03 leaves 4.
- Destroys: cell 99 and its own cells 67-69, 98; the accumulator returns the result.
- Cells: 19-98 (80).
- Instructions: worst 252, mean 119.9.
- Worst error: 2 in the last digit (thousandths) against the correctly rounded value, at x = 19; 405 of 999 inputs off.
- Algorithm: Meggitt's decimal pseudo-division in the form of Briggs's radix method: x doubled to 500..999 (exact), then multiplied by 1.1 and by 1.02 while it stays at most 1000, each factor's logarithm added to a sum held in two cells; the work is on the remainder 1000 - y (Meggitt's renormalised pseudo remainder, so three digits carry five places); what is left is its own logarithm up to a factor (Briggs's and Meggitt's tail rule); three-digit remainder, the 1.1 steps rounded.
- Sources:
  - Before 1968: H. Briggs, Arithmetica Logarithmica, London, 1624, chapter 14 (read in Ian Bruce's translation, and in D. Roegel's reconstruction, LOCOMAT, 2011); J. E. Meggitt, "Pseudo Division and Pseudo Multiplication Processes", IBM Journal of Research and Development 6(2), 210-226, 1962 (read in full), equations 7, 9 and 19; S. Newcomb, Logarithmic and Other Mathematical Tables, Holt, 1882, pp. 3, 9, 22 (read) (characteristic and mantissa).
  - After 1968, the same method in a calculator: D. S. Cochran, "Algorithms and Accuracy in the HP-35", Hewlett-Packard Journal, June 1972, 10-11 (read; after 1968).
- Status: worst error and misrounded count exact, by exhaustive check on HEARTBURN over the whole domain (999 inputs), called at these addresses through the convention, from one load, in two orders, and again moved down. No size is claimed minimal. The bound is a measurement: the error before the final rounding reaches 2.05 units. No routine of this kind is correctly rounded everywhere: 1000 ln x comes within 0.00034 of a half, so correct rounding needs six or seven digits before rounding, against the accumulator's four.
- Moving it: subtract d from every address that points into cells 19-98 (00 and 99 stay); it holds no address bases.

### lnx-a

1000 ln x, for x from 1 to 999, by pseudo-division with a four-digit remainder.

- Format and range:
  - Input: x, 1..999.
  - Result: round(1000 ln x), 0..6908, in the accumulator only (STO keeps the last three digits, SFT 03 leaves the thousands).
- Calling:

  ```
  cell  contents  notes
   03      108    CLA 08  x
   04      812    JMP 12  call lnx-a; JMP leaves the return jump in cell 99
   05      609    STO 09  the last three digits
   06      403    SFT 03  the thousands
   07      610    STO 10
   08      100    x = 100
   09      000    receives the last three digits
   10      000    receives the thousands
  ```

  The result is in the accumulator when cell 05 runs: 4605, more than a cell holds; STO keeps 605 and SFT 03 leaves 4.
- Destroys: cell 99 and its own cells 65-68, 98; the accumulator returns the result.
- Cells: 12-98 (87).
- Instructions: worst 283, mean 133.9.
- Worst error: 1 in the last digit (thousandths) against the correctly rounded value, at x = 4; 110 of 999 inputs off.
- Algorithm: Meggitt's decimal pseudo-division in the form of Briggs's radix method: x doubled to 500..999 (exact), then multiplied by 1.1 and by 1.02 while it stays at most 1000, each factor's logarithm added to a sum held in two cells; the work is on the remainder 1000 - y (Meggitt's renormalised pseudo remainder, so three digits carry five places); what is left is its own logarithm up to a factor (Briggs's and Meggitt's tail rule); four-digit remainder in the 1.1 steps (its tenths in a cell of their own), which divides their rounding error by ten.
- Sources:
  - Before 1968: H. Briggs, Arithmetica Logarithmica, London, 1624, chapter 14 (read in Ian Bruce's translation, and in D. Roegel's reconstruction, LOCOMAT, 2011); J. E. Meggitt, "Pseudo Division and Pseudo Multiplication Processes", IBM Journal of Research and Development 6(2), 210-226, 1962 (read in full), equations 7, 9 and 19; S. Newcomb, Logarithmic and Other Mathematical Tables, Holt, 1882, pp. 3, 9, 22 (read) (characteristic and mantissa).
  - After 1968, the same method in a calculator: D. S. Cochran, "Algorithms and Accuracy in the HP-35", Hewlett-Packard Journal, June 1972, 10-11 (read; after 1968).
- Status: worst error and misrounded count exact, by exhaustive check on HEARTBURN over the whole domain (999 inputs), called at these addresses through the convention, from one load, in two orders, and again moved down. No size is claimed minimal. The bound of one unit is also proven by argument: the integer model, which agrees with HEARTBURN on every input, has an error before the final rounding of at most 0.51 unit, and rounding is monotone, so the final error is at most 1. No routine of this kind is correctly rounded everywhere: 1000 ln x comes within 0.00034 of a half, so correct rounding needs six or seven digits before rounding, against the accumulator's four.
- Moving it: subtract d from every address that points into cells 12-98 (00 and 99 stay); it holds no address bases.

### alg-s

100 x 10^(m / 1000), the antilogarithm of a mantissa, by pseudo-multiplication.

- Format and range:
  - Input: a mantissa m, 0..999.
  - Result: round(100 x 10^(m / 1000)), 100..998: the three significant digits of the antilogarithm. A logarithm L = 1000c + m stands for (result) x 10^(c - 2).
- Calling:

  ```
  cell  contents  notes
   03      106    CLA 06  m
   04      812    JMP 12  call alg-s; JMP leaves the return jump in cell 99
   05      607    STO 07  the antilogarithm's three digits, from the accumulator
   06      301    m = 301
   07      000    receives the antilogarithm's three digits
  ```

  The result is in the accumulator when cell 05 runs: 200.
- Destroys: cell 99 and its own cells 33, 41, 52, 61-66, 98; the accumulator returns the result.
- Cells: 12-98 (87).
- Instructions: worst 911, mean 525.3.
- Worst error: 2 in the last digit (units of the third significant digit) against the correctly rounded value, at m = 965; 269 of 1,000 inputs off.
- Algorithm: Meggitt's decimal pseudo-multiplication with the tail first: the exponent is divided by the logarithms of 2, 1.1 and 1.01 while the product P (four digits in two cells) is multiplied by each factor, P + P / 10^k; what is left of the exponent is a factor 1 + cE applied by starting P at 1000 + cE, so the routine makes two passes, the first only to find E; the loop's SUB and SFT words are advanced from level to level and set back each pass; the exponent rescaled by ten from level to level, as Meggitt does (constants 301, 413, 432).
- Sources:
  - Before 1968: H. Briggs, Arithmetica Logarithmica, London, 1624, chapter 14 (read in Ian Bruce's translation, and in D. Roegel's reconstruction, LOCOMAT, 2011); J. E. Meggitt, "Pseudo Division and Pseudo Multiplication Processes", IBM Journal of Research and Development 6(2), 210-226, 1962 (read in full) (p. 221: the partial sum started at x).
  - After 1968, the tail-first start in the HP-35: J. Laporte, analyses of the HP-35 logarithm and e^x algorithms from its ROM, 1981 (French), 2005 (English) (read; after 1968); D. S. Cochran, "Algorithms and Accuracy in the HP-35", Hewlett-Packard Journal, June 1972, 10-11 (read; after 1968).
- Status: worst error and misrounded count exact, by exhaustive check on HEARTBURN over the whole domain (1,000 inputs), called at these addresses through the convention, from one load, in two orders, and again moved down. No size is claimed minimal. No routine of this kind is correctly rounded everywhere: 100 x 10^(m / 1000) comes within 0.00046 of a half, so correct rounding needs six or seven digits before rounding.
- Moving it: subtract d from every address that points into cells 12-98 (00 and 99 stay), address bases included: cell 53 (SUB 56).

### alg-a

100 x 10^(m / 1000), the antilogarithm of a mantissa, by pseudo-multiplication.

- Format and range:
  - Input: a mantissa m, 0..999.
  - Result: round(100 x 10^(m / 1000)), 100..998: the three significant digits of the antilogarithm. A logarithm L = 1000c + m stands for (result) x 10^(c - 2).
- Calling:

  ```
  cell  contents  notes
   03      106    CLA 06  m
   04      809    JMP 09  call alg-a; JMP leaves the return jump in cell 99
   05      607    STO 07  the antilogarithm's three digits, from the accumulator
   06      301    m = 301
   07      000    receives the antilogarithm's three digits
  ```

  The result is in the accumulator when cell 05 runs: 200.
- Destroys: cell 99 and its own cells 31, 34, 42, 53, 66-71, 98; the accumulator returns the result.
- Cells: 09-98 (90).
- Instructions: worst 943, mean 541.0.
- Worst error: 1 in the last digit (units of the third significant digit) against the correctly rounded value, at m = 1; 247 of 1,000 inputs off.
- Algorithm: Meggitt's decimal pseudo-multiplication with the tail first: the exponent is divided by the logarithms of 2, 1.1 and 1.01 while the product P (four digits in two cells) is multiplied by each factor, P + P / 10^k; what is left of the exponent is a factor 1 + cE applied by starting P at 1000 + cE, so the routine makes two passes, the first only to find E; the loop's SUB and SFT words are advanced from level to level and set back each pass; the constants kept in high and low parts (3010 as 3 and 10), the exponent at one scale.
- Sources:
  - Before 1968: H. Briggs, Arithmetica Logarithmica, London, 1624, chapter 14 (read in Ian Bruce's translation, and in D. Roegel's reconstruction, LOCOMAT, 2011); J. E. Meggitt, "Pseudo Division and Pseudo Multiplication Processes", IBM Journal of Research and Development 6(2), 210-226, 1962 (read in full) (p. 221: the partial sum started at x).
  - After 1968, the tail-first start in the HP-35: J. Laporte, analyses of the HP-35 logarithm and e^x algorithms from its ROM, 1981 (French), 2005 (English) (read; after 1968); D. S. Cochran, "Algorithms and Accuracy in the HP-35", Hewlett-Packard Journal, June 1972, 10-11 (read; after 1968).
- Status: worst error and misrounded count exact, by exhaustive check on HEARTBURN over the whole domain (1,000 inputs), called at these addresses through the convention, from one load, in two orders, and again moved down. No size is claimed minimal. No routine of this kind is correctly rounded everywhere: 100 x 10^(m / 1000) comes within 0.00046 of a half, so correct rounding needs six or seven digits before rounding.
- Moving it: subtract d from every address that points into cells 09-98 (00 and 99 stay), address bases included: cell 54 (SUB 58), cell 55 (SUB 61).

### exp-s

100 e^(v / 1000), for v from 0 to 2302, by pseudo-multiplication.

- Format and range:
  - Input: v, 0..2302, standing for x = v / 1000 < ln 10, in two parts: its last three digits and its thousands.
  - Result: round(100 e^x), 100..999: the three significant digits of e^x, which is 1..10.
  - Larger x: the caller subtracts 2303 from v and counts one more power of ten, at a relative error of 0.0004 each time (ln 10 = 2.302585).
- Calling:

  ```
  cell  contents  notes
   03      108    CLA 08  v's thousands
   04      653    STO 53  into exp-s's operand cell for them
   05      109    CLA 09  v's last three digits
   06      812    JMP 12  call exp-s; JMP leaves the return jump in cell 99
   07      610    STO 10  100 e^x, from the accumulator
   08      001    v = 1000, so x = 1: the thousands
   09      000    the last three digits
   10      000    receives 100 e^x
  ```

  The result is in the accumulator when cell 07 runs: 272.
- Destroys: cell 99 and its own cells 33, 41, 52, 62-67, 98; the accumulator returns the result. The thousands' operand cell 53 is kept, so a run of calls with one thousands digit stores it once.
- Cells: 12-98 (87).
- Instructions: worst 910, mean 524.8.
- Worst error: 1 in the last digit (units of the third significant digit) against the correctly rounded value, at v = 1; 522 of 2,303 inputs off.
- Algorithm: Meggitt's decimal pseudo-multiplication with the tail first: the exponent is divided by the logarithms of 2, 1.1 and 1.01 while the product P (four digits in two cells) is multiplied by each factor, P + P / 10^k; what is left of the exponent is a factor 1 + cE applied by starting P at 1000 + cE, so the routine makes two passes, the first only to find E; the loop's SUB and SFT words are advanced from level to level and set back each pass; the constants of ln: 0.693, 0.0953, 0.00995; the exponent rescaled by ten from level to level.
- Sources:
  - Before 1968: H. Briggs, Arithmetica Logarithmica, London, 1624, chapter 14 (read in Ian Bruce's translation, and in D. Roegel's reconstruction, LOCOMAT, 2011); J. E. Meggitt, "Pseudo Division and Pseudo Multiplication Processes", IBM Journal of Research and Development 6(2), 210-226, 1962 (read in full) (p. 221: the partial sum started at x).
  - After 1968, the tail-first start in the HP-35: J. Laporte, analyses of the HP-35 logarithm and e^x algorithms from its ROM, 1981 (French), 2005 (English) (read; after 1968); D. S. Cochran, "Algorithms and Accuracy in the HP-35", Hewlett-Packard Journal, June 1972, 10-11 (read; after 1968).
- Status: worst error and misrounded count exact, by exhaustive check on HEARTBURN over the whole domain (2,303 inputs), called at these addresses through the convention, from one load, in two orders, and again moved down. No size is claimed minimal. No routine of this kind is correctly rounded everywhere: 100 e^x comes within 0.00057 of a half, so correct rounding needs six or seven digits before rounding.
- Moving it: subtract d from every address that points into cells 12-98 (00 and 99 stay), address bases included: cell 54 (SUB 57).

## Not shipped

- None: every logarithm, antilogarithm and e^x routine built is shipped.

## Sources

Before 1968, so available to a CARDIAC programmer of the day:

- H. Briggs, Arithmetica Logarithmica, London, 1624, chapter 14 (read in Ian Bruce's translation, and in D. Roegel's reconstruction, LOCOMAT, 2011).
- S. Newcomb, Logarithmic and Other Mathematical Tables, Holt, 1882, pp. 3, 9, 22 (read).
- J. E. Meggitt, "Pseudo Division and Pseudo Multiplication Processes", IBM Journal of Research and Development 6(2), 210-226, 1962 (read in full).
- D. S. Cochran, "Internal programming of the 9100A calculator", Hewlett-Packard Journal, September 1968, 14-16 (read).
- D. W. Hagelbarger and S. Fingerman, CARDIAC: A Cardboard Illustrative Aid to Computation, Bell Telephone Laboratories, 1968.

After 1968:

- D. S. Cochran, "Algorithms and Accuracy in the HP-35", Hewlett-Packard Journal, June 1972, 10-11 (read).
- J. Laporte, analyses of the HP-35 logarithm and e^x algorithms from its ROM, 1981 (French), 2005 (English) (read).
