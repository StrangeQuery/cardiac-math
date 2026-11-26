# cardiac-math

Math routines for CARDIAC, the 1968 cardboard computer: integer arithmetic and elementary functions on its
three-digit decimal cells. Every routine is a listing in the style of the original manual. Copy the one you need
into your program.

| Folder | What | Default |
|---|---|---|
| [mul](mul/) | x * y, x * x | `umul-m`: 27 cells, at most 267 instructions |
| [div](div/) | x / y and x % y together, remainder alone, division and remainder by constants | `udvm-m`: 34 cells, at most 1,178 instructions |
| [sqrt](sqrt/) | integer square root, rounded down or to nearest | `uisq-s`: 18 cells, at most 322 instructions |
| [sign](sign/) | -x, \|x\| | `sneg-s`, `sabs-s` |
| [trig](trig/) | sin, sin and cos together, tan, arctangent, atan2, in whole degrees | `sin-m`: 72 cells, error 1 in 1000 |
| [log](log/) | log10, ln, antilog, e^x | `lgt-s`: 77 cells, error 1 in 1000 |

Each folder's README lists every variant with its range, cost and accuracy, says which is the default and when to
choose another, and shows how to call each one and where its answer ends up.

## Names

A name is three letters for the operation and a suffix for the variant. Integer routines also take a prefix for
signedness.

| Part | Meaning |
|---|---|
| `u` prefix | unsigned: non-negative operands only |
| `s` prefix | signed: operands from -999 to 999 |
| `-s` suffix | smallest |
| `-m` suffix | between the smallest and the fastest |
| `-f` suffix | fastest |
| `-a` suffix | most accurate, where the variants differ in accuracy rather than speed |

Routines for a fixed constant are named by operation and constant, for example `sdiv7` or `urem12`.

## Calling

CARDIAC defines no calling convention; these routines share one. Put any second operand in the routine's operand
cell, load the first into the accumulator, and JMP to the routine. JMP leaves the return jump in cell 99; the routine
copies it into its own last cell and finishes there, back at the instruction after your JMP. Constant routines are
inline code instead: no call.

## Moving a routine

Each routine is listed ending at cell 98. To move one down by k cells, subtract k from every address that refers to
one of its own cells, including the data words its README names as address bases. Cells 00 and 99, constants and
tables stay as they are.

## License

MIT. See [LICENSE](LICENSE).
