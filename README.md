# cardiac-math

Math routines for CARDIAC, the 1968 cardboard computer: integer arithmetic and elementary functions on its
three-digit decimal cells. Each routine is a listing in the style of the original manual, in a folder of its own
with a README giving its interface, cost, accuracy and sources. Copy the listing into your program.

## Names

A name is three letters for the operation and a suffix for the variant. Integer routines also take a prefix for
signedness: `udvm-s` is the smallest divmod of non-negative operands, `sin-f` the fastest sine. Every routine has a
suffix, even where it is the only variant.

| Part | Meaning |
|---|---|
| `u` prefix | unsigned: non-negative operands only |
| `s` prefix | signed: operands from -999 to 999 |
| `-s` suffix | smallest |
| `-f` suffix | fastest |
| `-m` suffix | between the smallest and the fastest |
| `-a` suffix | most accurate, where the variants differ in accuracy rather than speed |

| Operation | Computes |
|---|---|
| `neg` | -x |
| `abs` | \|x\| |
| `mul` | x * y |
| `sqr` | x * x |
| `dvm` | x / y and x % y together |
| `rem` | x % y alone |
| `isq` | floor(sqrt(x)) |
| `isr` | sqrt(x) to the nearest integer |
| `sin`, `scs` | 1000 sin x; sine and cosine from one routine with two entries |
| `tan` | 100 tan x |
| `atn`, `at2` | arctangent of a ratio; atan2(y, x), in whole degrees |
| `lgt`, `lnx` | 1000 log10 x; 1000 ln x |
| `alg`, `exp` | antilog of a mantissa; e^x |

## Calling convention

CARDIAC defines no calling convention. These routines share this one.

- The caller puts the second operand y, if there is one, in the routine's operand cell, loads the first operand x
 into the accumulator, and JMPs to the routine's first cell. JMP leaves the return jump in cell 99.
- The routine's first three cells are `STO x` (x into its own cell), `CLA 99` and `STO 98`: the return jump is
 copied into the routine's last cell before any jump of its own overwrites cell 99.
- The routine finishes by running into its last cell, which jumps back to the cell after the caller's JMP.
- The result is in the accumulator. A `dvm` routine's second result, the remainder, is in a named cell of the
 routine (its x cell).
- Every routine overwrites its own cells and cell 99. The `y` column below marks those that also change the
 operand cell; the others keep y, so a program that always uses the same y can store it once.
- Each routine is right on every call, whatever the accumulator and its own cells hold from the last call.
- Division truncates toward zero, and the remainder takes the dividend's sign, as Rust's `/` and `%` do.
- The sine-cosine routines have a second entry for the cosine; each README gives both.
- Instruction counts: the integer routines count from their first cell to the return jump; the elementary
  functions count from the caller's JMP, one more instruction. Each README says which.

## Placement

Each routine is listed so that its last cell is 98, and each listing's header gives its cell range. To move a
routine down by k cells, subtract k from every address that refers inside the routine: the address part of each
instruction that names one of the routine's cells, and of each data word that is such an instruction (the notes
call these "an instruction the routine copies and changes"). Cell numbers 00 and 99 stay as they are, and so do
the constants and tables. Two routines in one program can't both end at 98, so at most one stays where it is
listed.
The elementary functions also hold address bases: data words that are instructions with an address inside the
routine. Each README names them; they move with the routine.

## Status

- Correct: every routine was checked at its listed addresses, through the calling convention, on every input of its
  domain, called one input after another on one load, and again after moving it by the rule above.
- Size: a routine is called smallest only where an exhaustive search rules out every smaller routine in the stated
  model; otherwise its README gives the proven lower bound next to the shipped size. No size is claimed for the
  elementary functions.
- Speed: instruction counts are exact, worst and mean over the whole domain.
- Error: exact over the whole domain; for some functions a bound is also proven by argument, as the README says.

## Integer arithmetic

Worst and mean are instructions per call, measured over the whole domain. "Smallest" gives the size bound proven by
exhaustive search for the routine's core in the inline model (operands in cells, no call); no
bound is claimed for the called form.

| Routine | Computes | Domain | Cells | Worst / mean | y | Algorithm | Smallest? |
|---|---|---|---|---|---|---|---|
| [sneg-s](sign/#sneg-s) | -x | x from -999 to 999 | 8 (91-98) | 7 / 7.0 | - | SFT 04, then SUB x | no; inline -x of a cell is 3 cells, none of 2 |
| [sabs-s](sign/#sabs-s) | \|x\| | x from -999 to 999 | 10 (89-98) | 8 / 7.5 | - | sign test, x - x - x | no; inline \|x\| keeping x is 5 cells, none of 4 without SFT |
| [umul-s](mul/#umul-s) | x * y | y >= 0 and \|x * y\| <= 999, x of either sign | 17 (82-98) | 8,002 / 1,011.4 | y | repeated addition | no; inline 8 to 10 (none of 7 without SFT) |
| [umul-f](mul/#umul-f) | x * y | x, y >= 0 and x * y <= 999 | 39 (60-98) | 84 / 44.0 | y | multiplication by digit, computed jump | no; fastest measured |
| [usqr-s](mul/#usqr-s) | x * x | x from 0 to 31 | 17 (82-98) | 289 / 149.5 | - | sum of odd numbers | no; inline 8 to 11 (none of 7 without SFT) |
| [usqr-f](mul/#usqr-f) | x * x | x from 0 to 31 | 43 (56-98) | 9 / 9.0 | - | table of squares | no; fastest measured |
| [udvm-s](div/#udvm-s) | x / y and x % y | x from 0 to 999, y from 1 to 999 | 17 (82-98) | 8,002 / 36.3 | - | repeated subtraction | no; inline 8 to 10 (none of 7) |
| [udvm-f](div/#udvm-f) | x / y and x % y | x from 0 to 999, y from 1 to 999 | 38 (61-98) | 275 / 30.3 | - | decimal long division, Horner's rule | no; fastest measured |
| [urem-s](div/#urem-s) | x % y | x from 0 to 999, y from 1 to 999 | 11 (88-98) | 3,005 / 17.9 | - | repeated subtraction | yes, inline 5 cells, none of 4 |
| [sdvm-s](div/#sdvm-s) | x / y and x % y, signed | x and y from -999 to 999, y not 0 | 47 (52-98) | 8,021 / 49.8 | y | repeated subtraction on magnitudes | no; inline 8 to 40 (none of 7) |
| [sdvm-f](div/#sdvm-f) | x / y and x % y, signed | x and y from -999 to 999, y not 0 | 71 (28-98) | 304 / 50.3 | y | decimal long division on magnitudes | no; fastest measured |
| [srem-s](div/#srem-s) | x % y, signed | x and y from -999 to 999, y not 0 | 24 (75-98) | 3,013 / 22.7 | y | repeated subtraction or addition | no; inline 8 to 18 (none of 7) |
| [uisq-s](sqrt/#uisq-s) | floor(sqrt(x)) | x from 0 to 999 | 18 (81-98) | 322 / 217.8 | - | sum of odd numbers | no; inline 8 to 13 (none of 7) |
| [uisq-f](sqrt/#uisq-f) | floor(sqrt(x)) | x from 0 to 999 | 49 (50-98) | 100 / 32.4 | - | table start, sum of odd numbers | no; fastest measured |
| [uisr-s](sqrt/#uisr-s) | sqrt(x) to the nearest integer | x from 0 to 999 | 20 (79-98) | 302 / 203.5 | - | sum of even numbers | no; inline 7 to 15 (none of 6) |
| [uisr-f](sqrt/#uisr-f) | sqrt(x) to the nearest integer | x from 0 to 999 | 56 (43-98) | 105 / 37.4 | - | table start, odd numbers, rounding test | no; fastest measured |

## Elementary functions

| Name | Computes | Format | Cells | Instructions worst / mean | Worst error | Overwrites |
|---|---|---|---|---|---|---|
| [`sin-s`](sin-s/) | 1000 sin x | x 0..359 degrees; result -999..999, sin 90 = 999 | 33-98 (66) | 113 / 71.8 | 3 | 99, 64-65, 67, 72-73, 97-98 |
| [`sin-m`](sin-m/) | 1000 sin x | x 0..359 degrees; result -999..999, sin 90 = 999 | 27-98 (72) | 113 / 74.5 | 1 | 99, 59-60, 62, 66-67, 89, 98 |
| [`sin-f`](sin-f/) | 1000 sin x | x 0..359 degrees; result -999..999, sin 90 = 999 | 18-98 (81) | 53 / 46.5 | 1 | 99, 48-49, 52-53, 58-60, 89, 98 |
| [`scs-s`](scs-s/) | 1000 sin x and 1000 cos x | x 0..359 degrees; results -999..999, sin 90 = cos 0 = 999 | 32-98 (67) | sin 113 / 71.8; cos 114 / 73.0 | sin 3; cos 3 | 99, 64-65, 67, 72-73, 97-98 |
| [`scs-a`](scs-a/) | 1000 sin x and 1000 cos x | x 0..359 degrees; results -999..999, sin 90 = cos 0 = 999 | 22-98 (77) | sin 113 / 74.5; cos 116 / 77.8 | sin 1; cos 1 | 99, 58-59, 61, 66-67, 89, 98 |
| [`tan-s`](tan-s/) | 100 tan x | x 0..359 degrees; result -999..999, saturating at x mod 180 = 85..95 | 13-98 (86) | 21,114 / 15,903.7 | 1.362 | 99, 77-85, 97-98 |
| [`atn-s`](atn-s/) | atan(n / 100) | n -999..999 for n / 100; result -84..84 degrees | 26-98 (73) | 7,909 / 6,729.0 | 0.694 | 99, 83-90, 98 |
| [`atn-a`](atn-a/) | atan(n / 100) | n -999..999 for n / 100; result -84..84 degrees | 18-98 (81) | 10,110 / 8,169.4 | 0.567 | 99, 81-88, 98 |
| [`at2-s`](at2-s/) | atan2(y, x) | y, x -999..999, not both 0; result 0..359 degrees | 12-98 (87) | 7,925 / 7,140.2 | 0.699 | 99, 77-78, 80-85, 98 |
| [`lgt-s`](lgt-s/) | 1000 log10 x | x 1..999; result 1000 x characteristic + mantissa, 0..3000 | 22-98 (77) | 265 / 139.4 | 1 | 99, 69-71, 98 |
| [`lgt-a`](lgt-a/) | 1000 log10 x | x 1..999; result 1000 x characteristic + mantissa, 0..3000 | 15-98 (84) | 288 / 153.5 | 1 | 99, 67-70, 98 |
| [`lnx-s`](lnx-s/) | 1000 ln x | x 1..999; result 0..6908 | 19-98 (80) | 253 / 120.9 | 2 | 99, 67-69, 98 |
| [`lnx-a`](lnx-a/) | 1000 ln x | x 1..999; result 0..6908 | 12-98 (87) | 284 / 134.9 | 1 | 99, 65-68, 98 |
| [`alg-s`](alg-s/) | 100 x 10^(m / 1000) | mantissa m 0..999; result 100..998, three significant digits | 12-98 (87) | 912 / 526.3 | 2 | 99, 33, 41, 52, 61-66, 98 |
| [`alg-a`](alg-a/) | 100 x 10^(m / 1000) | mantissa m 0..999; result 100..998, three significant digits | 09-98 (90) | 944 / 542.0 | 1 | 99, 31, 34, 42, 53, 66-71, 98 |
| [`exp-s`](exp-s/) | 100 e^(v / 1000) | v 0..2302 (x = v / 1000 < ln 10) in two parts; result 100..999, three significant digits | 12-98 (87) | 911 / 525.8 | 1 | 99, 33, 41, 52, 62-67, 98 |

Errors: thousandths for sin, scs, lgt, lnx; hundredths for tan; degrees for atn and at2; the third significant digit for alg and exp.

| Name | Algorithm | Status | Place among the variants |
|---|---|---|---|
| `sin-s` | sine table, step 10, linear interpolation, entries by dynamic programming | error exhaustive | smallest of the sines; sin-m has error 1 for 6 more cells; sin-f is fastest |
| `sin-m` | sine table, step 5, quarter-wave symmetry, linear interpolation, entries by dynamic programming | error exhaustive | middle of the sines: error 1 like sin-f in 9 fewer cells, at sin-s's speed and 6 more cells than it |
| `sin-f` | as sin-m, interpolation by a computed jump into a chain of ADD | error exhaustive | fastest of the sines, at about half the instructions of sin-m and sin-s for 9 or 15 more cells |
| `scs-s` | sine table, step 10, linear interpolation, entries by dynamic programming | error exhaustive | smallest of the two-entry routines; scs-a has error 1 for 10 more cells |
| `scs-a` | sine table, step 5, quarter-wave symmetry, linear interpolation, entries by dynamic programming | error exhaustive | most accurate of the two-entry routines; scs-s is 10 cells smaller with error 3 |
| `tan-s` | partial-fraction tangent (Euler), exact floor(m a / b) by a Bresenham-style loop | error exhaustive; correct rounding everywhere proven out of reach | the only tangent shipped: the source's more accurate form (error 1.05 hundredths, floor(5731 / (90 + d)) and floor(27d / 77)) needs 93 cells as a subroutine and does not fit |
| `atn-s` | Horner polynomial arctangent, octant folding, Bresenham-style products | error exhaustive; correct rounding everywhere proven out of reach | smallest arctangent; atn-a is more accurate (0.567 against 0.694, 20 misrounded against 282) for 8 more cells and about 2,200 more instructions |
| `atn-a` | Horner polynomial arctangent, octant folding, Bresenham-style products | error exhaustive; correct rounding everywhere proven out of reach | most accurate arctangent; atn-s is 8 cells smaller and 2,200 instructions faster at worst. The three-term polynomial (worst 0.573, 86 misrounded, 4 cells fewer) lies between and is not shipped |
| `at2-s` | Horner polynomial arctangent, octant folding, Bresenham-style products | error exhaustive; correct rounding everywhere proven out of reach | the only atan2 that fits as a subroutine; the three-term form (worst 0.616 degrees) needs 91 cells |
| `lgt-s` | Meggitt's decimal pseudo-division (Briggs's radix method) | error exhaustive; correct rounding everywhere proven out of reach | smallest log10; lgt-a has the same worst error, correctly rounded more often (85% of inputs against 76%) and proven, for 7 more cells |
| `lgt-a` | Meggitt's decimal pseudo-division (Briggs's radix method) | error exhaustive; bound of 1 also proven by argument; correct rounding everywhere proven out of reach | most accurate log10 (same worst error as lgt-s, fewer misrounded inputs, bound proven); 7 cells more |
| `lnx-s` | Meggitt's decimal pseudo-division (Briggs's radix method) | error exhaustive; correct rounding everywhere proven out of reach | smallest ln; lnx-a has error 1 for 7 more cells |
| `lnx-a` | Meggitt's decimal pseudo-division (Briggs's radix method) | error exhaustive; bound of 1 also proven by argument; correct rounding everywhere proven out of reach | most accurate ln; lnx-s is 7 cells smaller with error 2 |
| `alg-s` | Meggitt's decimal pseudo-multiplication, tail first | error exhaustive; correct rounding everywhere proven out of reach | smallest antilog; alg-a has error 1 for 3 more cells |
| `alg-a` | Meggitt's decimal pseudo-multiplication, tail first | error exhaustive; correct rounding everywhere proven out of reach | most accurate antilog; alg-s is 3 cells smaller and faster with error 2 |
| `exp-s` | Meggitt's decimal pseudo-multiplication, tail first | error exhaustive; correct rounding everywhere proven out of reach | the only e^x |

Not shipped: the step-10 fast sine (72 cells, error 3) and the separate cosine routines of the source, which the two-entry routines cover; the more accurate tangent and the three-term atan2, which do not fit as subroutines; the three-term arctangent, between atn-s and atn-a.

### Tables against calculator methods

- CORDIC, the method of the HP-35 (Volder 1959; decimal by Meggitt 1962), loses on this machine: it needs a guard digit that three-digit cells lack. A decimal CORDIC sine with one-cell x and y is off by 15 units; with a four-digit x or y it is 2 units in 86 to 93 cells and 1,160 to 1,660 instructions, against sin-m's 1 unit in 72 cells and 113 instructions.
- Decimal CORDIC arctangent in three-digit arithmetic is off by 6.9 degrees, the HP-35 tangent by 59 hundredths.
- A short table read is a built CLA, four cells and four or five instructions: for a function sampled at a few dozen points a table beats any routine here by an order of magnitude.

## Sources

Before 1968, so available to a CARDIAC programmer of the day:

- [TAYLOR] Taylor, A. E. IBM 1401, 1440 and 1460 programming and operating techniques. IBM Technical Information Exchange TIE 5-0021, 1964 (IBM internal). Section H-5: "the repetitive-addition method of multiplication".
- [NICOMACHUS] Nicomachus of Gerasa. Introduction to Arithmetic, about 100 AD: the squares as sums of the odd numbers (secondhand).
- [ANDREE] Andree, R. V. Programming the IBM 650 Magnetic Drum Computer and Data-Processing Machine. Holt, Rinehart and Winston, 1958. pp. 49-53: "division is accomplished by repeated subtraction, just as it is on a desk calculator".
- [IBM650] IBM. 650 Magnetic Drum Data-Processing Machine, Manual of Operation. Form 22-6060-1, 1955. p. 36: division by repeated subtraction with shifting, "essentially the same as the paper and pencil process"; pp. 36-37: the remainder has the sign of the dividend.
- [IBM1401] IBM. Programs for IBM 1401 Card Systems, Preliminary Specifications. J24-0209-2, 1960. pp. 13-14: the Divide subroutine, by repetitive subtraction with a shift factor aligning divisor and dividend; the remainder has the sign of the dividend.
- [IBM1620] IBM. IBM 1620 Data Processing System Reference Manual. Form A26-4500-2, 1961. p. 21: the remainder has the sign of the dividend.
- [PRESTON] IBM. IBM 1401 Programs, catalog C20-1601-5, 1962. Program 1401-03.0.014 (B. Preston): "square root subroutine using odd integer method".
- [COCHRAN] Cochran, D. S. Internal programming of the 9100A calculator. Hewlett-Packard Journal 20(1), September 1968, pp. 14-16. p. 15: the square root "is done by pseudo division" from the sum of the odd numbers.
- [COLLINS] Collins, G. E. Square root subroutine. IBM 650 Library Program Abstract, 1956. In Additional Abstracts of IBM 650 Programs, March 1957: a table look-up followed by two Newton iterations.
- H. Briggs, Arithmetica Logarithmica, London, 1624, chapter 14 (read in Ian Bruce's translation, and in D. Roegel's reconstruction, LOCOMAT, 2011).
- L. Euler, the partial-fraction expansion of the tangent, eighteenth century (secondhand).
- S. Newcomb, Logarithmic and Other Mathematical Tables, Holt, 1882, pp. 3, 9, 22 (read).
- The Aryabhatiya, A.D. 499: 24 sines at steps of 3 3/4 degrees (secondhand, through Gupta 1967).
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
- J. Laporte, analyses of the HP-35 logarithm and e^x algorithms from its ROM, 1981 (French), 2005 (English) (read).
