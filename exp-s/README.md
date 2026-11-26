# exp-s

100 e^(v / 1000), for v from 0 to 2302, by pseudo-multiplication.

- Input: v, 0..2302, standing for x = v / 1000 < ln 10, in two parts: its last three digits and its thousands.
- Result: round(100 e^x), 100..999: the three significant digits of e^x, which is 1..10.
- Larger x: the caller subtracts 2303 from v and counts one more power of ten, at a relative error of 0.0004 each time (ln 10 = 2.302585).
- Calling: v's thousands in cell 53, its last three digits in the accumulator, JMP 12; result in the accumulator.
- Cells: 12-98 (87).
- Overwrites: cell 99, and its own cells 33, 41, 52, 62-67, 98; no other cell.
- Moving: subtract d from every instruction address that points into cells 12-98 (00 and 99 stay), and from the address bases: cell 54 (SUB 57).
- Instructions (calling JMP to return, inclusive): worst 911, mean 525.8.
- Worst error: 1 in the last digit (units of the third significant digit) against the correctly rounded value, at v = 1; 522 of 2,303 inputs off.
- Algorithm: Meggitt's decimal pseudo-multiplication with the tail first: the exponent is divided by the logarithms of 2, 1.1 and 1.01 while the product P (four digits in two cells) is multiplied by each factor, P + P / 10^k; what is left of the exponent is a factor 1 + cE applied by starting P at 1000 + cE, so the routine makes two passes, the first only to find E; the loop's SUB and SFT words are advanced from level to level and set back each pass; the constants of ln: 0.693, 0.0953, 0.00995; the exponent rescaled by ten from level to level.
- Sources:
 - Before 1968: H. Briggs, Arithmetica Logarithmica, London, 1624, chapter 14 (read in Ian Bruce's translation, and in D. Roegel's reconstruction, LOCOMAT, 2011); J. E. Meggitt, "Pseudo Division and Pseudo Multiplication Processes", IBM Journal of Research and Development 6(2), 210-226, 1962 (read in full) (p. 221: the partial sum started at x).
 - After 1968, the tail-first start in the HP-35: J. Laporte, analyses of the HP-35 logarithm and e^x algorithms from its ROM, 1981 (French), 2005 (English) (read; after 1968); D. S. Cochran, "Algorithms and Accuracy in the HP-35", Hewlett-Packard Journal, June 1972, 10-11 (read; after 1968).
- Status: the worst error and the misrounded count are exact, by exhaustive check on HEARTBURN over the whole domain (2,303 inputs), calling the routine at these addresses through the convention, from one load, in two orders, and again moved down by the rule above. Instruction counts are exact measurements. No claim is made that the size is minimal.
- No routine of this kind is correctly rounded everywhere: 100 e^x comes within 0.00057 of a half, so correct rounding needs six or seven digits before rounding.
- Variant: the only e^x.
- Listing: `exp-s.cardiac`, the routine alone.
