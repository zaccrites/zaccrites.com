---
title: Software Package Design
date: 2022-08-22
draft: true
tags: [programming]
# slug: magical-world-of-sat-solvers-part-2
---

Package Design
Packages should have as narrow a scope as possible. Follow the single-responsibility principle: a package should only ever have one, and only one, reason to change. Service packages should ideally contain a single executable with a specific job. Library packages should deal with a single system or system component.

This concept of limited scope applies to more than just packages. A module, class, or function should also have a single responsibility, in order to help manage complexity. If you have to use the word “and” to describe the purpose of a function or class, take that as a hint that the function or class may be doing too much. It should be split into pieces with individual responsibilities.

Import Structure
Do not force library package users to import names from a deeply nested hierarchy.

# service/__main__.py
from grid_nbstatus.loaders.xml import XmlLoader
from grid_nbstatus.loaders.live import load_live
Instead, re-export these names at the package level:

# grid_nbstatus/__init__.py
from grid_nbstatus.loaders.xml import XmlLoader
from grid_nbstatus.loaders.live import load_live

__all__ = ["XmlLoader", "load_live"]


# service/__main__.py
from grid_nbstatus import XmlLoader, load_live
Be sure to order the elements of __all__ logically, as it is the same order that the names will appear in auto-generated documentation. Entries should be sorted roughly in priority order, with primary API functions and classes appearing at the top of the list and less important types appearing at the bottom. For entries of equal priority, sort alphabetically.

If external code should not use a package function or class directly, do not export it in __init__.py.
