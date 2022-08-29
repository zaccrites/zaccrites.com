---
title: "The Magical World of SAT Solvers (Part 2): Sudoku"
date: 2022-08-25
draft: true
tags: [python, tools]
slug: magical-world-of-sat-solvers-part-2
series: [sat-solvers]
description: 'Using "true or false" logic statements to solve puzzles and riddles'
icon: "fa-solid fa-table-cells"
---

In [Part 1]({{< ref "blog/2022/sat_solvers_part_1.md" >}}),
we covered the basics of boolean algebra and SAT Solvers
and learned how to solve a Sudoku puzzle using a boolean logic expression.

This time we will look at another famous puzzle
and a riddle from a famous children's book.



## Sudoku

[Sudoku](https://en.wikipedia.org/wiki/Sudoku) puzzles are a perfect example.
The puzzle takes place on a 9x9 grid,
into which the solver must enter numbers with the following rules:

1. Each row must contain the numbers 1-9.
2. Each column must contain the numbers 1-9.
3. Each 3x3 subgrid must contain the numbers 1-9.

The puzzle starts with some of the boxes pre-filled,
and it is up to the solver to find the values of the rest of the boxes.

TODO: put image or table here

{{< highlight text >}}
7 _ _   _ _ _   9 _ _
_ 8 _   _ 4 6   _ _ 7
_ _ _   _ 2 _   _ _ _

_ _ _   3 _ _   _ _ _
_ _ 1   _ _ _   _ 5 _
_ 4 _   _ 7 8   _ _ 9

_ _ _   2 _ _   _ _ 6
_ _ 4   _ 6 3   1 _ _
_ 3 _   9 _ _   _ _ _
{{< / highlight >}}


Let's write a Python program to solve this particularly difficult Sudoku puzzle.

{{< highlight python >}}
import pycosat

raw_board = """
7 _ _   _ _ _   9 _ _
_ 8 _   _ 4 6   _ _ 7
_ _ _   _ 2 _   _ _ _

_ _ _   3 _ _   _ _ _
_ _ 1   _ _ _   _ 5 _
_ 4 _   _ 7 8   _ _ 9

_ _ _   2 _ _   _ _ 6
_ _ 4   _ 6 3   1 _ _
_ 3 _   9 _ _   _ _ _
""".strip()

board = [
    [
        None if cell == "_" else int(cell)
        for cell in row.split()
    ]
    for row in raw_board.splitlines()
    if row
]

clauses = []

{{< / highlight >}}

## Translating Constraints to Logic Expressions

The first step is to define variables corresponding to properties of the puzzle.
In this case, the only properties of note are the dimensions of the grid
and the value of the cells therein.
It is not possible to represent nine distinct values in a 1-bit boolean variable,
so instead define nine variables-- one for each possible value of the cell.

$$
R^k_{i,j}
$$

Where $k$ is the value of the cell,
$i$ is the cell column, and $j$ is the cell row.

For example, the cell in row 2, column 5 is known to contain a `4`,
so the variable $R^4\_{5,2}$ is `true`.
The others for that cell,
$R^1\_{5,2}$, $R^2\_{5,2}$, $R^3\_{5,2}$, $R^5\_{5,2}$, $R^6\_{5,2}$, $R^7\_{5,2}$, $R^8\_{5,2}$, and $R^9\_{5,2}$ are all `false`.

With nine possible values for each cell and 81 cells on a 9x9 grid,
the system has 729 total variables.

{{< highlight python >}}
import itertools

variables = {}  # variable numbers from (i, j, k)
numbers = {}  # (i, j, k) from variable number

# Positive literal
def r(i, j, k, *, value=True):
    key = (i, j, k)
    number = variables.setdefault(key, len(variables) + 1)
    numbers[number] = key
    return number if value else -number

# Negative literal
def nr(i, j, k):
    return r(i, j, k, value=False)

# Helper to iterate over board rows, board columns,
# or possible cell values
def itercoords(axes=1):
    if axes == 1:
        return range(9)
    return itertools.product(*[range(9) for _ in range(axes)])

{{< / highlight >}}

### Known Cell Values

For cells with a given known value $k\_0$,
assert the corresponding variable:

$$
R^{k\_0}_{i,j}
$$

De-assert the rest:

$$
\neg R^{k}_{i,j} \qquad k \neq k\_0
$$

{{< highlight python >}}
for i, j in itercoords(2):
    if (k0 := board[j][i]) is not None:
        for k in itercoords():
            clauses.append([r(i, j, k)] if k == k0 else [nr(i, j, k)])
{{< / highlight >}}

A single-term clause means that the variable must have the specified value,
as there is no other way to satisfy that clause.

### Unknown Cell Values

There are nine variables for each potential cell value,
but obviously only one of these can be `true` in the solved board.
This constraint has two parts.
The first is to declare that at least one of the values must be selected.

$$
R^1_{i,j} \vee R^1_{i,j} \vee ... \vee R^9_{i,j}
$$

{{< highlight python >}}
for i, j in itercoords(2):
    clauses.append([r(i, j, k) for k in itercoords()])
{{< / highlight >}}

The second is to declare that no two can be set at once.
That is, either can one can be selected (or neither),
but both values cannot be selected simultaneously.

$$
(\neg R^{k\_1}\_{i,j} \vee \neg R^{k\_2}\_{i,j}) \qquad k\_1 \neq k\_2
$$

{{< highlight python >}}
for i, j in itercoords(2):
    for k1, k2 in itercoords(2):
        if k1 != k2:
            clauses.append([nr(i, j, k1), nr(i, j, k2)])
{{< / highlight >}}

Taken together, these rules define the constraint that *exactly one*
value must be set for each cell.


### Rows, Columns, and Subgrids

Another way of saying that a row must contain every number
is that no two cells in a row can have the same number.
This is similar to the rule that a cell can contain only one number.

$$
(\neg R^k\_{i\_1,j} \vee \neg R^k\_{i\_2,j}) \qquad i\_1 \neq i\_2
$$

{{< highlight python >}}
for j, k in itercoords(2):
    for i1, i2 in itercoords(2):
        if i1 != i2:
            clauses.append([nr(i1, j, k), nr(i2, j, k)])
{{< / highlight >}}

The same is true for columns:

$$
(\neg R^k\_{i,j\_1} \vee \neg R^k\_{i,j\_2}) \qquad j\_1 \neq j\_2
$$

{{< highlight python >}}
for i, k in itercoords(2):
    for j1, j2 in itercoords(2):
        if j1 != j2:
            clauses.append([nr(i, j1, k), nr(i, j2, k)])
{{< / highlight >}}

And it is true for 3x3 subgrids as well:

$$
(\neg R^k\_{i\_1,j\_1} \vee \neg R^k\_{i\_2,j\_2}) \qquad (i\_1, j\_1) \neq (i\_2, j\_2)
$$

{{< highlight python >}}
# Find the position of each subgrid
def iter_subgrid_coords():
    return itertools.product(*[range(0, 9, 3) for _ in range(2)])

# Find the positions inside a subgrid
def iter_subgrid_inner_coords(i0, j0):
    return itertools.product(
        (i0 + offset for offset in range(3)),
        (j0 + offset for offset in range(3)),
    )

# Find distinct subgrid
def iter_distinct_subgrid_coord_pairs(i0, j0):
    for i1, j1 in iter_subgrid_inner_coords(i0, j0):
        for i2, j2 in iter_subgrid_inner_coords(i0, j0):
            if (i1, j1) != (i2, j2):
                yield i1, j1, i2, j2

for i0, j0 in iter_subgrid_coords():
    for i1, j1, i2, j2 in iter_distinct_subgrid_coord_pairs(i0, j0):
        for k in itercoords():
            clauses.append([nr(i1, j1, k), nr(i2, j2, k)])
{{< / highlight >}}


## Result

With all of the constraints defined,
the SAT solver makes short work of the puzzle.

{{< highlight python >}}
solution = pycosat.solve(clauses)

solved_board = [[None] * 9 for _ in range(9)]

for variable in solution:
    if variable > 0:
        i, j, k = numbers[variable]
        solved_board[j][i] = str(k + 1)

for j, row in enumerate(solved_board, 1):
    for i, cell in enumerate(row, 1):
        print(cell, end="   " if i % 3 == 0 else " ")
    print("\n" if j % 3 == 0 else "")
{{< / highlight >}}

{{< highlight text >}}
Before                         After
7 _ _   _ _ _   9 _ _          7 6 2   8 3 5   9 4 1
_ 8 _   _ 4 6   _ _ 7          3 8 9   1 4 6   5 2 7
_ _ _   _ 2 _   _ _ _          4 1 5   7 2 9   6 3 8

_ _ _   3 _ _   _ _ _          9 2 8   3 5 1   7 6 4
_ _ 1   _ _ _   _ 5 _          6 7 1   4 9 2   8 5 3
_ 4 _   _ 7 8   _ _ 9          5 4 3   6 7 8   2 1 9

_ _ _   2 _ _   _ _ 6          1 5 7   2 8 4   3 9 6
_ _ 4   _ 6 3   1 _ _          8 9 4   5 6 3   1 7 2
_ 3 _   9 _ _   _ _ _          2 3 6   9 1 7   4 8 5
{{< / highlight >}}

The solution's variables are set exactly as we said they must be.
Each cell contains a single value, and that value is unique in its
row, column, and 3x3 subgrid.
What happens if we remove the row- and column-uniqueness constraints?

{{< highlight text >}}
7 5 3   9 5 3   9 5 2
9 8 2   8 4 6   8 4 7
6 4 1   7 2 1   6 3 1

9 6 3   3 5 2   8 4 2
8 5 1   9 4 1   7 5 1
7 4 2   6 7 8   6 3 9

9 6 2   2 7 4   9 7 6
8 5 4   8 6 3   1 5 3
7 3 1   9 5 1   8 4 2
{{< / highlight >}}

Each 3x3 subgrid still has all nine numbers in it,
but rows and columns no longer do.

The SAT solver is free to return *any* solution that satisfies the given expression. If the constraints are not defined well enough, it may find an incorrect one, a far as the problem is concerned.


