# sin-m

1000 sin x, x in whole degrees, from a table every 5 degrees.

- Input: x, a whole number of degrees, 0..359.
- Result: round(1000 sin x), halves away from zero, held to -999..999.
- The cap: STO keeps three digits and would store 1000 as 000, so 1000 is held to 999. sin 89, 90, 91 give 999 and sin 269, 270, 271 give -999; the reference is held the same way.
- Calling: x in the accumulator, JMP 27; result in the accumulator.
- Cells: 27-98 (72).
- Overwrites: cell 99, and its own cells 59-60, 62, 66-67, 89, 98; no other cell.
- Moving: subtract d from every instruction address that points into cells 27-98 (00 and 99 stay), and from the address bases: cell 65 (ADD 69).
- Instructions (calling JMP to return, inclusive): worst 113, mean 74.5.
- Worst error: 1 in the last digit (thousandths) against the correctly rounded value, at x = 1; 88 of 360 inputs off.
- Algorithm: sine table with quarter-wave symmetry and linear interpolation; 19 entries of cos 5k over 0..90 degrees, chosen by dynamic programming to minimise the worst interpolated error rather than rounded; the doubled angle gives index and fraction by SFT 01 and SFT 33; table reads are built instructions; D r added in a loop.
- Sources:
 - Before 1968: the Aryabhatiya, A.D. 499: 24 sines at steps of 3 3/4 degrees (secondhand, through Gupta 1967); R. C. Gupta, "Bhaskara I's approximation to sine", Indian Journal of History of Science 2(2), 121-136, 1967 (read).
 - Library sine routines of the day (polynomials, not tables): IBM, Additional Abstracts of IBM 650 Programs, March 1957, abstracts 3.1.011 and 3.1.021 (read); Royal McBee, LGP-30 Subroutine Manual, October 1960, Program 14.0, sine-cosine subroutine in degrees (read).
 - Choosing table entries by dynamic programming for the interpolation: no earlier source known; none was searched for.
- Status: the worst error and the misrounded count are exact, by exhaustive check on HEARTBURN over the whole domain (360 inputs), calling the routine at these addresses through the convention, from one load, in two orders, and again moved down by the rule above. Instruction counts are exact measurements. No claim is made that the size is minimal.
- The table's entries are optimal for this interpolation among entries within 4 units of the rounded sines (exact dynamic program over the entries).
- Variant: middle of the sines: error 1 like sin-f in 9 fewer cells, at sin-s's speed and 6 more cells than it.
- Listing: `sin-m.cardiac`, the routine alone.
