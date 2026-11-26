# tan-s

100 tan x, x in whole degrees, by partial fractions.

- Input: x, a whole number of degrees, 0..359.
- Result: round(100 tan x) where it fits a cell, x mod 180 = 0..84 and 96..179 (-951..951).
- Saturation: 999 for x mod 180 = 85..90, -999 for 91..95.
- Calling: x in the accumulator, JMP 13; result in the accumulator.
- Cells: 13-98 (86).
- Overwrites: cell 99, and its own cells 77-85, 97-98; no other cell.
- Moving: subtract d from every instruction address that points into cells 13-98 (00 and 99 stay), and from the address bases: cell 89 (ADD 85), cell 90 (SUB 85).
- Instructions (calling JMP to return, inclusive): worst 21,114, mean 15,903.7.
- Worst error: 1.362 hundredths from the exact value, at x = 54; 124 of 360 inputs not correctly rounded.
- Algorithm: partial-fraction tangent (Euler): 100 tan d = A / (90 - d) - A / (90 + d) + R(d), A = 18000 / pi; computed as floor(5730 / (90 - d)) - floor(5730 / (90 + d)) + floor(6d / 17), constants by exhaustive search over the forms the multiply computes; each term an exact floor(m a / b) by a Bresenham-style error-accumulation loop that stores nothing larger than b; d mod 180 folded to 0..90, the sign put back by a stored ADD or SUB.
- Sources:
 - Before 1968: L. Euler, the partial-fraction expansion of the tangent, eighteenth century (secondhand); J. E. Bresenham, "Algorithm for computer control of a digital plotter", IBM Systems Journal 4(1), 1965 (secondhand: the loop is the standard one).
- Status: the worst error and the misrounded count are exact, by exhaustive check on HEARTBURN over the whole domain (360 inputs), calling the routine at these addresses through the convention, from one load, in two orders, and again moved down by the rule above. Instruction counts are exact measurements. No claim is made that the size is minimal.
- Proven impossible here: correct rounding everywhere. 100 tan d is within 0.008 of a half at d = 2, 18 and 68, so no formula in three- and four-digit arithmetic rounds it correctly at every d, and an exact table (41 cells) does not fit beside its code.
- Variant: the only tangent shipped: the source's more accurate form (error 1.05 hundredths, floor(5731 / (90 + d)) and floor(27d / 77)) needs 93 cells as a subroutine and does not fit.
- Listing: `tan-s.cardiac`, the routine alone.
