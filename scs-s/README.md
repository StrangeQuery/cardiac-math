# scs-s

1000 sin x and 1000 cos x, x in whole degrees, from one table every 10 degrees, with an entry for each.

- Input: x, a whole number of degrees, 0..359.
- Result: round(1000 sin x) or round(1000 cos x), halves away from zero, held to -999..999.
- The cap: STO keeps three digits and would store 1000 as 000, so 1000 is held to 999. sin 89-91 and cos 359, 0, 1 give 999; sin 269-271 and cos 179-181 give -999; the reference is held the same way.
- Calling: x in the accumulator; JMP 33 for the sine, JMP 32 for the cosine; result in the accumulator.
- Cells: 32-98 (67).
- Overwrites: cell 99, and its own cells 64-65, 67, 72-73, 97-98; no other cell.
- Moving: subtract d from every instruction address that points into cells 32-98 (00 and 99 stay), and from the address bases: cell 71 (ADD 75).
- Instructions, sin (calling JMP to return, inclusive): worst 113, mean 71.8.
- Instructions, cos (calling JMP to return, inclusive): worst 114, mean 73.0.
- Worst error, sin: 3 in the last digit (thousandths) against the correctly rounded value, at x = 60; 196 of 360 inputs off.
- Worst error, cos: 3 in the last digit (thousandths) against the correctly rounded value, at x = 3; 196 of 360 inputs off.
- Algorithm: sine table with linear interpolation; the angle mapped to b = 0..180 with sin x = sin(b - 90), so the table covers -90..90 degrees and no sign is restored; 19 entries at a step of 10 degrees, chosen by dynamic programming to minimise the worst interpolated error (as CARDIAC computes it, truncation included) rather than rounded; index and fraction by SFT 01 and SFT 33; table reads are built instructions; D r added in a loop, the sum offset by -700 to stay within a cell; the cosine as sin(x + 90): one ADD in front of the sine's entry.
- Sources:
 - Before 1968: the Aryabhatiya, A.D. 499: 24 sines at steps of 3 3/4 degrees (secondhand, through Gupta 1967); R. C. Gupta, "Bhaskara I's approximation to sine", Indian Journal of History of Science 2(2), 121-136, 1967 (read).
 - Library sine routines of the day (polynomials, not tables): IBM, Additional Abstracts of IBM 650 Programs, March 1957, abstracts 3.1.011 and 3.1.021 (read); Royal McBee, LGP-30 Subroutine Manual, October 1960, Program 14.0, sine-cosine subroutine in degrees (read).
 - Choosing table entries by dynamic programming for the interpolation: no earlier source known; none was searched for.
- Status: the worst error and the misrounded count are exact, by exhaustive check on HEARTBURN over the whole domain (360 inputs, each entry), calling the routine at these addresses through the convention, from one load, in two orders, and again moved down by the rule above. Instruction counts are exact measurements. No claim is made that the size is minimal.
- The table's entries are optimal for this interpolation among entries within 4 units of the rounded sines (exact dynamic program over the entries).
- Variant: smallest of the two-entry routines; scs-a has error 1 for 10 more cells.
- Listing: `scs-s.cardiac`, the routine alone.
