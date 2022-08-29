---
title: ClearCase is Awesome
date: 2022-08-22T00:30:45-06:00
draft: true
tags: [programming, version-control]
# slug: magical-world-of-sat-solvers-part-2
series: []
description: "Looking back at the best version control system I ever used"
# icon: "fa-solid fa-code-branch"
icon: "fa-solid fa-dumpster-fire"
DisableDollarMath: true
---

Git is overrated.

Why should I use Git in a world where ClearCase exists?

You can use ClearCase from the command line *and* from a GUI!
Does Git have a GUI? Didn't think so. Oh wait, it does? There are many, catering to different use cases? Hmm.

Well I bet it doesn't have version tree!
The ClearCase version tree GUI is amazing.
It shows a node for every version of the file that ever existed
on any branch,
and it is truly awe-inspring when you see the version tree for an ancient file in the codebase.
Yessir, after waiting a mere five minutes for the graph to finish rendering you too can experience this amazing sight.
It doesn't matter that a few of the thousands upon thousands of displayed versions aren't relevant to anyone anymore, let alone a developer who wasn't even born yet when the file was originally created.
Have some respect!

And that's another thing! Git is so impersonal.
Atomic changesets of the entire project?
Why would I want that?
This is *version* control for *versions* of *files*.
I want to know what the history of *this file* is in isolation.
The history of other files changed alongside it is irrelevant,
and I bet that because Git doesn't store per-file histories
it can't tell me what every committed version of a file was.
What's that? `git show` can show the contents of a file at any revision without forcing me to wait for the entire version tree render?

Okay, but what about getting the code in the first place?
Does Git have *snapshot views*?
Nothing gets me more excited at the start of a new task
than updating my snapshot view.
Watching that progress bar tick by fills me up with such anticipation.
One minute, two, three go by, and eventually
my snapshot view update finishes and I can start working.
What's a mere ten minute wait to get the *latest and greatest*
code?
What's that? `git pull` runs in less than a minute?
Less than a *second*? No, that can't be right.

Okay so you have "snapshot views", but do you have *dynamic* views?
ClearCase does! You can mount your entire view as a network share!
I've heard that compilers actually perform better that way.
Do you have any *idea* how slow mechanical hard drives are?
There's a gigabit NIC sitting in my computer just begging to fetch files from a remote ClearCase VOB whenever I need them.
Plus if you're working on a shared branch with someone you can bond over fixing merge conflicts with them in *realtime*.

That's the other great thing about ClearCase.
At *my* company we always kept branches around.
Creating a new branch was a momentous occasion.
I got to see the great history of branches from all time
load one by one into the branch management GUI.
Only after they all filed in, and my respects were paid,
was I able, nay, *permitted* to add my own branch to this sacred ledger.
Git makes creating a branch so easy that it's almost not worth doing.
*Nothing worth doing is easy*, that's what my parents always told me.
I've heard some people create temporary branches *in the middle of developing a feature*. Such decadence.

With ClearCase, the developers are part of a *community*.
You really feel the tribal love when you go to delete or move a file
in your snapshot view and ClearCase takes the time to tell you that you can't. Someone is *using* that file, you see.
It has a checkout in someone else's snapshot view!
It doesn't matter that that person left the company months ago and is no longer around to undo the checkout so you can progress with your task. That colleague is still around *in spirit*, and it would be disrespectful to remove the checkout that he left behind as a token by which we can remember him.

In ClearCase we take the time to actually *tell* our fellow developers that we're working on a file.
We *check out* the file, even leaving a checkout *message* so others know what we're doing.
We even have options! We can use an *unreserved* checkout to indicate to others that our changes are likely to be minimal, so it's okay to work on the same file at the same time.
Or we can use a *reserved* checkout, to prevent other developers from working on that file at all!
It's important to be *considerate* with these things,
otherwise we might get angry at eachother for misusing checkouts!
I would sure hate to be allowed to check out a file if another developer was working on that file.
ClearCase saves me from this potentially embarressing faux pau.
Git allows it. Git doesn't even *understand* the idea of a reserved checkout.

Speaking of things that are worth something,
hasn't anyone ever told these Git lunatics that things given aren't worth anything?
Git is *free*. They give it away for nothing!
If it was worth something they would charge for it!
That's how I know ClearCase is amazing.
If it wasn't amazing then why would IBM charge so much for it?
In fact they charge so much that they won't even tell you unless you call them first.
They're modest like that.
But word on the street is that it's in the $100k - $1M dollar range, depending on how many developers you're hooking up by buying it.
