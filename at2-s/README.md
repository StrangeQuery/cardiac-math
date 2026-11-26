# at2-s

atan2(y, x), the direction of (x, y) in whole degrees.

- Input: y and x, -999..999, not both 0.
- Result: the direction of (x, y) anticlockwise from the positive x axis, whole degrees, rounded, 0..359 (360 is 0).
- Calling: x in cell 79, y in the accumulator, JMP 12; result in the accumulator.
- Cells: 12-98 (87).
- Overwrites: cell 99, and its own cells 77-78, 80-85, 98; no other cell.
- Moving: subtract d from every instruction address that points into cells 12-98 (00 and 99 stay); there are no address bases.
- Instructions (calling JMP to return, inclusive): worst 7,925, mean 7,140.2.
- Worst error: 0.699 degrees from the exact value, at y = -912, x = -890; 462,024 of 3,996,000 inputs not correctly rounded.
- Algorithm: Horner polynomial arctangent with octant folding: a = min(|x|, |y|), b = max, t = atan(a / b) = z (604 - 152 z) tenths of a degree, each product an exact floor(m a / b) by a Bresenham-style loop; then the reflections 90 - t, -t, 180 - t and + 360.
- Sources:
 - Before 1968: C. Hastings, Jr., Approximations for Digital Computers, Princeton University Press, 1955 (not obtained; its arctangent forms read as reprinted in Abramowitz and Stegun); M. Abramowitz and I. A. Stegun (eds.), Handbook of Mathematical Functions, National Bureau of Standards, Applied Mathematics Series 55, 1964 (read in part), 4.4.47-4.4.48, p. 81; J. E. Bresenham, "Algorithm for computer control of a digital plotter", IBM Systems Journal 4(1), 1965 (secondhand: the loop is the standard one).
 - The coefficients are this library's own fits, not Hastings's.
- Status: the worst error and the misrounded count are exact, by exhaustive check on HEARTBURN over the whole domain (3,996,000 inputs), calling the routine at these addresses through the convention, from one load, in two orders, and again moved down by the rule above. Instruction counts are exact measurements. No claim is made that the size is minimal.
- Proven impossible here: correct rounding. Over 0 <= p <= q <= 999 the ratio p / q comes within 8.9 x 10^-8 of a rounding boundary tan(j - 0.5 degrees) (170 / 613 and 15.5 degrees), so a correctly rounded atan2 needs the ratio to about eight significant digits.
- Variant: the only atan2 that fits as a subroutine; the three-term form (worst 0.616 degrees) needs 91 cells.
- Listing: `at2-s.cardiac`, the routine alone.
