# atn-s

atan(n / 100), in whole degrees, by a polynomial of two terms.

- Input: n, -999..999, standing for the ratio n / 100.
- Result: atan(n / 100) in whole degrees, rounded, -84..84.
- Calling: n in the accumulator, JMP 26; result in the accumulator.
- Cells: 26-98 (73).
- Overwrites: cell 99, and its own cells 83-90, 98; no other cell.
- Moving: subtract d from every instruction address that points into cells 26-98 (00 and 99 stay); there are no address bases.
- Instructions (calling JMP to return, inclusive): worst 7,909, mean 6,729.0.
- Worst error: 0.694 degrees from the exact value, at n = -780; 282 of 1,999 inputs not correctly rounded.
- Algorithm: Horner polynomial arctangent with octant folding: z = min(|n|, 100) / max(|n|, 100), t = atan z in tenths of a degree as a polynomial in z, each product an exact floor(m a / b) by a Bresenham-style loop; coefficients are minimax fits tuned by coordinate descent on the integer pipeline; then 90 - t for |n| > 100 and the sign; two terms, z (604 - 152 z).
- Sources:
 - Before 1968: C. Hastings, Jr., Approximations for Digital Computers, Princeton University Press, 1955 (not obtained; its arctangent forms read as reprinted in Abramowitz and Stegun); M. Abramowitz and I. A. Stegun (eds.), Handbook of Mathematical Functions, National Bureau of Standards, Applied Mathematics Series 55, 1964 (read in part), 4.4.47-4.4.48, p. 81; J. E. Bresenham, "Algorithm for computer control of a digital plotter", IBM Systems Journal 4(1), 1965 (secondhand: the loop is the standard one).
 - The coefficients are this library's own fits, not Hastings's.
- Status: the worst error and the misrounded count are exact, by exhaustive check on HEARTBURN over the whole domain (1,999 inputs), calling the routine at these addresses through the convention, from one load, in two orders, and again moved down by the rule above. Instruction counts are exact measurements. No claim is made that the size is minimal.
- Proven impossible here: a correctly rounded atan(n / 100) needs a table of its 84 thresholds (41 cells packed) plus about 25 cells to walk it, more than the machine leaves.
- Variant: smallest arctangent; atn-a is more accurate (0.567 against 0.694, 20 misrounded against 282) for 8 more cells and about 2,200 more instructions.
- Listing: `atn-s.cardiac`, the routine alone.
