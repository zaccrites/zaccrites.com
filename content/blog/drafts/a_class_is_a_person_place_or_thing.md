---
title: A Class is a Person, Place, or Thing
date: 2022-08-22
draft: true
tags: [programming, object-oriented-programming]
# slug: magical-world-of-sat-solvers-part-2
icon: "fa-solid fa-sitemap"
---

{{< quote speaker="A Noun is a Person, Place, or Thing" source="Schoolhouse Rock!" style="em" speakerStyle="quote" speakerLink="https://www.youtube.com/watch?v=929mBQPQB3g" >}}
Well every person you can know,\
And every place that you can go,\
And anything that you can show, \
You know they're nouns.

A noun's a special kind of word,\
It's any name you ever heard,\
I find it quite interesting,\
A noun's a person, place, or thing.
{{< / quote >}}

Classes should represent a thing, not an action.
Classes are nouns, not verbs.
If a class name ends with "er" and has a single public method,
there's a good chance that a free function or closure would be better.

Even objects which exist only for use in
[RAII resource management]({{< ref "blog/drafts/raii.md" >}})
should represent things.
The classic example is a mutex lock.
That's a *noun* lock, not the *verb* lock.
