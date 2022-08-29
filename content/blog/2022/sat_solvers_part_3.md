---
title: "The Magical World of SAT Solvers (Part 3): Snape's Riddle"
date: 2022-08-27
draft: true
tags: [python, tools]
slug: magical-world-of-sat-solvers-part-3
series: [sat-solvers]
description: "Using a SAT solver to solve Snape's riddle from *Harry Potter and the Sorceror's Stone*"
icon: "fa-solid fa-wand-sparkles"
---

## Snape's Riddle

In the penultimate chapter of J. K. Rowling's *Harry Potter and the Sorcerer's Stone*,
the protagonists Harry, Ron, and Hermione must complete a sequence of challenges
to prevent the villain, Voldemort, from obtaining the titular stone.
After one such challenge disables Ron, Harry and Hermione continue on to the next
and encounter a sequence of potion bottles of various sizes and a note.
Flames block both the door leading to the next room and back to the previous.
The only way out is to solve the riddle posed by the creator of the challenge:
the school's potions master, Snape.

{{< quote source="Harry Potter and the Sorceror's Stone" style="em" link="" subsource="J.K. Rowling" >}}
Danger lies before you, while safety lies behind,\
Two of us will help you, whichever you would find,\
One among us seven will let you move ahead,\
Another will transport the drinker back instead,\
Two among our number hold only nettle wine,\
Three of us are killers, waiting hidden in line.\
Choose, unless you wish to stay here for evermore,\
To help you in your choice, we give you these clues four:\
First, however slyly the poison tries to hide\
You will always find some on nettle wine’s left side;\
Second, different are those who stand at either end,\
But if you would move onwards, neither is your friend;\
Third, as you see clearly, all are different size,\
Neither dwarf nor giant holds death in their insides;\
Fourth, the second left and the second on the right\
Are twins once you taste them, though different at first sight.\
{{< / quote >}}


In the book, Hermione quickly solves the riddle,
locating both potions to move through the flames.
How might we solve this riddle using a SAT solver?

### Variables

We can derive the following properties of the puzzle:

1. There are seven potion bottles.
2. One bottle moves the drinker forward to the next challenge.
3. One bottle moves the drinker backward to the previous room.
4. Two bottles hold harmless, but useless, nettle wine.
5. Three of the bottles contain poison.
6. A bottle containing poison will always appear to the left of a bottle containing wine.
7. The leftmost and rightmost bottles do not contain the forward-moving potion.
8. Of the small, medium, and large bottle sizes, small and large bottles will not contain poison.
9. The first bottle in from the left and right ends have the same contents,
   but different sizes.

We create a variable for the properties of each bottle with position $i$,
where $i = 0$ is the leftmost bottle and $i = 6$ is the rightmost:

* Whether the bottle contains the forward-moving potion, $F\_i$
* Whether the bottle contains the backward-moving potion, $B\_i$
* Whether the bottle contains nettle wine, $W\_i$
* Whether the bottle contains poison, $P\_i$
* Whether the bottle is small, $S\_i$
* Whether the bottle is medium-sized, $M\_i$
* Whether the bottle is large, $L\_i$

### Constraints

#### Each bottle contains exactly one kind of liquid

Each bottle contains a kind of liquid.
$$ F\_i \vee B\_i \vee P\_i \vee W\_i $$

But not two at the same time.
$$ \neg F\_i \vee \neg B\_i $$
$$ \neg F\_i \vee \neg P\_i $$
$$ \neg F\_i \vee \neg W\_i $$
$$ \neg B\_i \vee \neg P\_i $$
$$ \neg B\_i \vee \neg W\_i $$
$$ \neg P\_i \vee \neg W\_i $$

#### Each bottle has exactly one size

A bottle can be small, medium, or large
$$ S\_i \vee M\_i \vee L\_i $$

But not two sizes at the same time.
$$ \neg S\_i \vee \neg M\_i $$
$$ \neg S\_i \vee \neg L\_i $$
$$ \neg M\_i \vee \neg L\_i $$

#### Exactly one bottle contains the forward- and backward-moving potions

Any one of the bottles might have them:

$$ F\_0 \vee F\_1 \vee F\_2 \vee F\_3 \vee F\_4 \vee F\_5 \vee F\_6 $$
$$ B\_0 \vee B\_1 \vee B\_2 \vee B\_3 \vee B\_4 \vee B\_5 \vee B\_6 $$

But no two bottles have them:

$$ \neg F\_i \vee \neg F\_j \qquad i \neq j $$
$$ \neg B\_i \vee \neg B\_j \qquad i \neq j $$

#### Two bottles contain wine

{{< quote >}}
Two among our number hold only nettle wine,
{{< / quote >}}

For every distinct triplet of bottles $(k\_0, k\_1, k\_2)$,
add a clause which asserts that at least one of them does not contain wine.
This disqualifies solutions with more than two bottles of wine.

$$
\neg W\_{k\_0} \vee \neg W\_{k\_1} \vee \neg W\_{k\_2}
\qquad k\_0 \neq k\_1 \neq k\_2
$$

For every distinct group of six bottles $(k\_0,...,k\_5)$,
add a clause which asserts that at least one of them *does* contain wine.
This disqualifies solutions with less than two bottles of wine.

$$
W\_{k\_0} \vee W\_{k\_1} \vee W\_{k\_2} \vee W\_{k\_3} \vee W\_{k\_4} \vee W\_{k\_5}
\qquad k\_0 \neq k\_1 \neq k\_2 \neq k\_3 \neq k\_4 \neq k\_5
$$

In general, asserting that groups of $k + 1$ have at least one variable de-asserted
and groups of $n - k + 1$ have at least one variable asserted ensures that
exactly $k$ variables are asserted out of a group of $n$.
Removing one of these two constraints can be used to ensure that
*at least* or *at most* $k$ variables are asserted.

#### Three bottles contain poison

{{< quote >}}
Three of us are killers, waiting hidden in line.
{{< / quote >}}

This rule is implied, because with seven bottles,
two containing wine, one each containing the forward-
and backward-moving potions,
there are only three left to contain poison.

It is possible to apply the same strategy for the poison as the wine
(disqualifying solutions with more or less than three bottles of poison),
but it is not necessary.

#### Poison always appears to the left of wine

{{< quote >}}
First, however slyly the poison tries to hide\
You will always find some on nettle wine’s left side;
{{< / quote >}}

$W\_i$ implies $P\_{i - 1}$.
That is, a wine bottle at $i$ requires a poison bottle at $i - 1$.

$$
\neg W\_i \vee P\_{i - 1}
$$

#### The bottles at the ends have different contents, and do not contain the forward potion

{{< quote >}}
Second, different are those who stand at either end,\
But if you would move onwards, neither is your friend;
{{< / quote >}}

Neither contain the forward potion.

$$
\neg F\_0
$$

$$
\neg F\_6
$$

Neither contain the same liquid.

$$
\neg B\_0 \vee \neg B\_6
$$

$$
\neg P\_0 \vee \neg P\_6
$$

$$
\neg W\_0 \vee \neg W\_6
$$



#### Small and large bottles do not contain poison

{{< quote >}}
Neither dwarf nor giant holds death in their insides;
{{< / quote >}}

$S\_i$ and $L\_i$ imply $\neg P\_i$.
That is, a bottle can be small (or large), or contain poison,
but not both at the same time.

$$
\neg S\_i \vee \neg P\_i
$$

$$
\neg L\_i \vee \neg P\_i
$$


#### The second-from-left and second-from-right bottles have the same contents, but different sizes

{{< quote >}}
Fourth, the second left and the second on the right
Are twins once you taste them, though different at first sight.
{{< / quote >}}

If they have the same contents,
then they can't contain the forward- or backward-moving potions,
of which there is only one bottle containing each.
If one contains poison, they both do.
Otherwise, both contain wine.

$$ P\_1 \vee W\_1 $$
$$ P\_5 \vee W\_5 $$
$$ \neg P\_1 \vee \neg W\_5 $$
$$ \neg W\_1 \vee \neg P\_5 $$

The bottles are different sizes.

$$ \neg S\_1 \vee \neg S\_5 $$
$$ \neg M\_1 \vee \neg M\_5 $$
$$ \neg L\_1 \vee \neg L\_5 $$



### The Solution

{{< highlight text >}}
Bottle 0 is size Medium and contains Poison
Bottle 1 is size Large and contains Wine
Bottle 2 is size Medium and contains Poison
Bottle 3 is size Large and contains Forward-moving Potion
Bottle 4 is size Medium and contains Poison
Bottle 5 is size Medium and contains Wine
Bottle 6 is size Large and contains Backward-moving Potion
{{< / highlight >}}

Great! We can safely drink Bottle 3 and move onward!
...or can we?

{{< highlight python >}}
solutions = list(pycosat.itersolve(clauses))
print("Found", len(solutions), "solution" if len(solutions) == 1 else "solutions")
# Found 108 solutions
{{< / highlight >}}

Uh-oh. Looks like we're missing some constraints.
Let's look at the riddle again:

{{< quote >}}
Third, as you see clearly, all are different size,
{{< / quote >}}

Harry and Hermione might be able to see the bottles clearly,
but we the readers cannot, and our heroes didn't think to tell us.
As it turns out, the riddle as presented in the book is inconclusive.
[A person working out the answer](https://expatronum.wordpress.com/solving-snapes-logic-puzzle/)
can narrow it down to two possibilities.
If we were to add these additional inferred constraints
then the SAT solver could narrow it down to the two possibilities as well.
Without actually seeing the bottles, it is impossible to know the answer for sure.

Years after the publication of *Harry Potter and the Sorceror's Stone*,
the "Pottermore" companion website released an illustration of the scene,
which finally gives us a look at the sizes of the bottles.

TODO: add image
https://expatronum.files.wordpress.com/2020/04/img_4491.jpg?w=460

We can add these as constraints.

$$ M\_0 \wedge M\_1 \wedge S\_2 \wedge M\_3 \wedge M\_4 \wedge L\_5 \wedge M\_6 $$

{{< highlight python >}}
# Define bottle sizes
for i, size in enumerate([M, M, S, M, M, L, M]):
    clauses.append([r(i, size)])
{{< / highlight >}}

And the new solution?

{{< highlight text >}}
Bottle 0 is size Medium and contains Poison
Bottle 1 is size Medium and contains Wine
Bottle 2 is size Small and contains Forward-moving Potion
Bottle 3 is size Medium and contains Poison
Bottle 4 is size Medium and contains Poison
Bottle 5 is size Large and contains Wine
Bottle 6 is size Medium and contains Backward-moving Potion
Found 1 solution
{{< / highlight >}}

It's a good thing we didn't drink Bottle 3.
With the additional information the SAT solver found the one correct solution,
as described in the book.

> Hermione read the paper several times. Then she walked up and down the line of bottles,
> muttering to herself and pointing at them. At last, she clapped her hands.
> "Got it," she said. "The smallest bottle will get us through the black fire – towards the Stone."
> Harry looked at the tiny bottle.
> "There’s only enough there for one of us," he said. "That's hardly one swallow."
> They looked at each other.
> "Which one will get you back through the purple flames?"
> Hermione pointed at a rounded bottle at the right end of the line.

## Conclusion

TODO
