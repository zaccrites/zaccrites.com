---
title: "The Magical World of SAT Solvers (Part 1): The Eight Queens Puzzle"
date: 2022-08-23
draft: true
tags: [python, tools]
slug: magical-world-of-sat-solvers
series: [sat-solvers]
description: 'Using "true or false" logic statements to solve puzzles and riddles'
icon: "fa-solid fa-chess"
---

TODO: introduction

TODO: create nicer sudoku board images

<!--more-->

If you don't feel like learning about boolean algebra right now
and just want to see how this could possibly apply to Harry Potter,
[click here]({{< ref "blog/2022/sat_solvers_part_3.md" >}})
to skip right to it.

## Boolean Algebra

The mathematical rules describing logical values and operations which can be performed on them
is called [**boolean algebra**](https://en.wikipedia.org/wiki/Boolean_algebra).
Variables in this system can have only two values, `true` and `false`,
and there are three main operations:

* **Conjunction** ("and"), denoted $\wedge$,
  where the expression $A \wedge B$ is `true` if and only if both $A$ *and* $B$ are `true`
* **Disjunction** ("or"), denoted $\vee$,
  where the expression $A \vee B$ is `true` if and only if either $A$ *or* $B$, or both, are `true`
* **Negation** ("not"), denoted $\neg$,
  where the expression $ \neg A $ is `true` if and only if $A$ is not `true` (that is, $A$ is `false`)

These operations can be combined to produce more complex behavior.
For example, the following is an expression for the "exclusive or" operation,
which is true if and only if $A$ or $B$ is `true`, but not both.

$$
A \veebar B = (A \wedge \neg B) \vee (\neg A \wedge B)
$$


## Satisfiability

A boolean logic expression is said to be *satisfiable*
if there exists some combination of values for its variables
which can make the expression `true`.
For example, the expression $A \wedge \neg B$ is satisfied
by assigning $A$ and $B$ the values `true` and `false`
respectively. $A \wedge \neg A$, however, is *unsatisfiable* because no matter the value of $A$,
the expression can never be true.

Proving that an arbitrary boolean logic expression is satisfiable
is known as the
[boolean satisfiability problem](https://en.wikipedia.org/wiki/Boolean_satisfiability_problem),
or "SAT". Small expressions with a few variables and clauses
are simple enough even for a person to determine,
but the problem gets exponentially more difficult when the number of variables
and clauses gets into the hundreds or thousands, as often happens in real-world
SAT problems.

## SAT Solvers

A SAT solver is a computer program designed to answer the boolean satisfiability problem.
It takes a boolean logic expression as input and either finds an example solution
which proves that the expression is satisfiable,
or it can determine that the expression is unsatisfiable, or "UNSAT".
They use sophisticated algorithms and heuristics to search a potentially huge problem space ($n$ variables means there are $2^n$ possible solutions to check) extremely quickly.

SAT solvers typically require that the boolean logic expression input
come in [**conjunctive normal form**](https://en.wikipedia.org/wiki/Conjunctive_normal_form),
which means that the expression is a conjunction of clauses, where each clause is a disjunction of "literals".
A **literal** is the atomic unit of boolean logic expressions,
taking the form of the positive of a variable (e.g., $A$),
or its negation (e.g., $\neg B$).
For example, the following expression has four variables
and five literals:

$$
(A \vee B) \wedge (C \vee D) \wedge (\neg C \vee \neg B)
$$

Each variable is assigned a number, and each clause appears on its own line.
Positive literals get a positive numbers,
while negative literals get negative numbers.
In this example, $A$, $B$, $C$, and $D$ correspond to the numbers
1, 2, 3, and 4 respectively.

{{< highlight text >}}
1 2
3 4
-3 -2
{{< / highlight >}}

### SAT in Python

[pycosat](https://pypi.org/project/pycosat/) is a Python binding to the
[PicoSAT](http://fmv.jku.at/picosat/) solver.
We can use it in Python code directly to solve SAT problems:

{{< highlight python >}}
import pycosat

clauses = [
    [1, 2],
    [3, 4],
    [-3, -2],
]
print(pycosat.solve(clauses))  # [1, -2, -3 , 4]
{{< / highlight >}}

* $A$ = `true`
* $B$ = `false`
* $C$ = `false`
* $D$ = `true`

The solver found a solution which satisfies all three clauses of the expression above.

## Solving Problems

At their heart, SAT solvers are constraint solvers.
Many problems can be described completely by defining a set of binary constraints.



## The Eight Queens Puzzle

The "eight queens" puzzle challenges the player to place eight queens on a chess board
such that no two attack each other.
This is another constraint problem to which a SAT solver is perfectly suited.

In chess, the queen is undoubtedly the most powerful piece.
It can move as far as it likes along its rank (row), file (column),
or either diagonal.
Therefore we have four constraints to model:

1. The board must have eight queens.
2. No two queens may share a rank
3. No two queens may share a file
4. No two queens may share a diagonal

This time each square has only one variable:
whether a queen sits on it or not.

{{< highlight python >}}
import pycosat
import itertools

board_size = 8
clauses = []

def itercoords(axes=1):
    if axes == 1:
        return range(board_size)
    return itertools.product(*[range(board_size) for _ in range(axes)])

coords_to_numbers = {}
numbers_to_coords = {}

def r(i, j, value=True):
    try:
        number = coords_to_numbers[(i, j)]
    except KeyError:
        number = coords_to_numbers[(i, j)] = len(coords_to_numbers) + 1
        numbers_to_coords[number] = (i, j)
    return number if value else -number

def nr(*args):
    return r(*args, value=False)

{{< / highlight >}}


### Rank and File

With eight queens on the board and no two sharing a rank or file,
constraint #1 comes for free with #2 or #3.
"No two queens may share a rank or file"
is another way of saying
"each rank and file must have exactly one queen".
Therefore if a cell in rank `j` or file `i` contains a queen,
no others can.

{{< highlight python >}}
# Each rank must have exactly one queen
for j in itercoords():
    clauses.append([r(i, j) for i in itercoords()])
for j, i1, i2 in itercoords(3):
    if i1 != i2:
        clauses.append([nr(i1, j), nr(i2, j)])

# Each file must have exactly one queen
for i in itercoords():
    clauses.append([r(i, j) for j in itercoords()])
for i, j1, j2 in itercoords(3):
    if j1 != j2:
        clauses.append([nr(i, j1), nr(i, j2)])
{{< / highlight >}}


### Diagonals

Diagonals may have at most one queen, but are not required to have one.

{{< highlight python >}}
diagonals = set()

for x0 in itercoords():
    # Moving up and to the left, where (0, 0) is the top-left corner.
    diagonals.add(frozenset(
        (x0 + offset, offset) for offset in itercoords()
        if 0 <= x0 + offset < board_size
    ))
    diagonals.add(frozenset(
        (offset, x0 + offset) for offset in itercoords()
        if 0 <= x0 + offset < board_size
    ))
    # Moving up and to the right
    diagonals.add(frozenset(
        (x0 - offset, offset) for offset in itercoords()
        if 0 <= x0 - offset < board_size
    ))
    diagonals.add(frozenset(
        (board_size - (x0 - offset) - 1, board_size - offset - 1)
        for offset in itercoords()
        if 0 <= board_size - (x0 - offset) - 1 < board_size
    ))

diagonals = {diagonal for diagonal in diagonals if len(diagonal) > 1}

# Each diagonal may have at most one queen
for diagonal in diagonals:
    for (i1, j1), (i2, j2) in itertools.product(diagonal, diagonal):
        if (i1, j1) != (i2, j2):
            clauses.append([nr(i1, j1), nr(i2, j2)])
{{< / highlight >}}

### The Solution

Now all that's left to do is hand it off to the SAT solver.

{{< highlight python >}}
solution = pycosat.solve(clauses)
queen_squares = {
    numbers_to_coords[number]
    for number in solution
    if number > 0
}

print("   " + " ".join(str(i) for i in itercoords()))
for j in itercoords():
    row = [
        "*" if (i, j) in queen_squares else " "
        for i in itercoords()
    ]
    print(f"{j} |" + "|".join(row) + "|")
{{< / highlight >}}

And the solution is:

{{< highlight text >}}
   0 1 2 3 4 5 6 7
0 | | | | | | | |*|
1 | | | |*| | | | |
2 |*| | | | | | | |
3 | | |*| | | | | |
4 | | | | | |*| | |
5 | |*| | | | | | |
6 | | | | | | |*| |
7 | | | | |*| | | |
{{< / highlight >}}

... but is that the only solution?
Surely with so many ways of arranging eight pieces on a 64-square grid,
there must be another position which satisfies the rules of the puzzle?

As it turns out, there are 92 solutions for a 8x8 board.
The SAT solver doesn't find them all because we didn't ask it to.
Remember, all it does is find a single example to prove that the
boolean logic expression is satisfiable.

### Finding All Solutions

We can force the solver to find another solution by
adding a clause specifically banning the solution it found.

{{< highlight python >}}
from datetime import datetime

start = datetime.now()
solutions = []
while solution != "UNSAT":
    solutions.append(solution)
    clauses.append([-number for number in solution])
    solution = pycosat.solve(clauses)
duration = (datetime.now() - start).total_seconds() * 1000

print("Found", len(solutions), f"solutions in {duration:.03f} ms")
# Found 92 solutions in 147.997 ms

{{< / highlight >}}

By inverting the variables,
we're telling the solver that it that specific combination of literals isn't a solution.
By repeating this process with every solution the solver finds,
we can find all possible solutions.
When all of the solutions have been banned,
the expression becomes unsatisfiable.

Incidentally, `pycosat` has a convenience method for this process:

{{< highlight python >}}
start = datetime.now()
solutions = list(pycosat.itersolve(clauses))
duration = (datetime.now() - start).total_seconds() * 1000
print("Found", len(solutions), f"solutions in {duration:.03f} ms")
# Found 92 solutions in 3.001 ms
{{< / highlight >}}

`solveiter` is nearly fifty times faster than manually banning solutions!
[The reason](https://github.com/conda/pycosat#implementation-of-itersolve)
has more to do with Python overhead than the SAT solver itself.
The manual version requires that the Python bindings convert the "list-of-numbers"
clauses into the native conjunction-normal form (CNF) representation.
`itersolve` does the conversion once and stays within the C extension code as long as possible.

### Results on Different Board Sizes

Board Size | Solutions | `itersolve` (ms) | Manual (ms)
---------- | --------- | ---------------- | -----------
1          | 1         | 0.000            | 0.000
2          | UNSAT     | N/A              | N/A
3          | UNSAT     | N/A              | N/A
4          | 2         | 0.000            | 0.000
5          | 10        | 0.000            | 1.001
6          | 4         | 1.003            | 1.001
7          | 40        | 1.002            | 20.002
8          | 92        | 3.759            | 149.510
9          | 352       | 19.99            | 1807.176
10         | 724       | 92.022           | 19252.204
11         | 2680      | 1592.669         | N/A
12         | 14200     | 22541.537        | N/A


As it turns out, there are no solutions for boards smaller than 4x4.
Technically a 1x1 "board" with a queen on its single square is a solution,
but that feels like cheating.
A 2x2 or 3x3 board ensures that the queens will share a rank, file, or diagonal.

{{< highlight text >}}
   0      0 1         0 1 2
0 |*|     0 | |*|     0 | | |*|
          1 |*| |     1 | |*| |
                      2 |*| | |

{{< / highlight >}}

The two solutions to a 4x4 board are mirror images of one another.

{{< highlight text >}}
   0 1 2 3         0 1 2 3
0 | | |*| |     0 | |*| | |
1 |*| | | |     1 | | | |*|
2 | | | |*|     2 |*| | | |
3 | |*| | |     3 | | |*| |
{{< / highlight >}}

The solutions increase exponentially with board size
(except for a 6x6 board, which has only four solutions)

The number of known solutions for boards up to 27x27
is documented in [OEIS A000170](https://oeis.org/A000170),
at 234,907,967,154,122,528 solutions.




## Conclusion

TODO: conclusion

In the [next part]({{< ref "blog/2022/sat_solvers_part_2.md" >}}),
we'll look at another classic
constraint solution puzzle, and how a young wizard
might just have needed a SAT solver had he not
had his clever friend with him to find the answer the old fashioned way.
