# lgt-a

1000 log10 x, for x from 1 to 999, by pseudo-division with a four-digit remainder.

- Input: x, 1..999.
- Result: round(1000 log10 x), 0..3000, as tables of logarithms lay it out: 1000 x characteristic + mantissa. STO keeps the mantissa; SFT 03 leaves the characteristic. log10 200 gives 2301.
- For x = 999 the correctly rounded result is 3000: characteristic 3, mantissa 000.
- Calling: x in the accumulator, JMP 15; result in the accumulator.
- Cells: 15-98 (84).
- Overwrites: cell 99, and its own cells 67-70, 98; no other cell.
- Moving: subtract d from every instruction address that points into cells 15-98 (00 and 99 stay); there are no address bases.
- Instructions (calling JMP to return, inclusive): worst 288, mean 153.5.
- Worst error: 1 in the last digit (thousandths) against the correctly rounded value, at x = 11; 145 of 999 inputs off.
- Algorithm: Meggitt's decimal pseudo-division in the form of Briggs's radix method: x doubled to 500..999 (exact), then multiplied by 1.1 and by 1.01 while it stays at most 1000, each factor's logarithm added to a sum held in two cells; the work is on the remainder 1000 - y (Meggitt's renormalised pseudo remainder, so three digits carry five places); what is left is its own logarithm up to a factor (Briggs's and Meggitt's tail rule); four-digit remainder in the 1.1 steps (its tenths in a cell of their own), which divides their rounding error by ten.
- Sources:
 - Before 1968: H. Briggs, Arithmetica Logarithmica, London, 1624, chapter 14 (read in Ian Bruce's translation, and in D. Roegel's reconstruction, LOCOMAT, 2011); J. E. Meggitt, "Pseudo Division and Pseudo Multiplication Processes", IBM Journal of Research and Development 6(2), 210-226, 1962 (read in full), equations 7, 9 and 19; S. Newcomb, Logarithmic and Other Mathematical Tables, Holt, 1882, pp. 3, 9, 22 (read) (characteristic and mantissa).
 - After 1968, the same method in a calculator: D. S. Cochran, "Algorithms and Accuracy in the HP-35", Hewlett-Packard Journal, June 1972, 10-11 (read; after 1968).
- Status: the worst error and the misrounded count are exact, by exhaustive check on HEARTBURN over the whole domain (999 inputs), calling the routine at these addresses through the convention, from one load, in two orders, and again moved down by the rule above. Instruction counts are exact measurements. No claim is made that the size is minimal.
- The bound of one unit is also proven by argument: the integer model, which agrees with HEARTBURN on every input, has an error before the final rounding of at most 0.64 unit, and rounding is monotone, so the final error is at most 1.
- No routine of this kind is correctly rounded everywhere: 1000 log10 x comes within 0.00086 of a half, so correct rounding needs six or seven digits before rounding, against the accumulator's four.
- Variant: most accurate log10 (same worst error as lgt-s, fewer misrounded inputs, bound proven); 7 cells more.
- Listing: `lgt-a.cardiac`, the routine alone.
