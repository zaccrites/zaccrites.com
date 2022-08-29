---
title: Rock-SOLID Design
date: 2022-08-22
draft: true
tags: [programming, object-oriented-programming]
# slug: magical-world-of-sat-solvers-part-2
---


In object-oriented programming, there exists a set of five design principles which make computer program designs simpler to understand, more flexible in operation, and easier to maintain. These principles lead to techniques which can manage complexity as programs grow over time.

While these principles were originally intended to manage complexity in object-oriented programming environments, they are not limited to that domain. These ideas can help manage complexity in any environment where many functional pieces work together to achieve a goal. Distributing the required complexity among those pieces using the right interfaces is as much an art as a science. When done right, software maintenance can be fast, easy, and even enjoyable. When done wrong, software maintenance is slow, difficult, and utterly painful.

The SOLID Principles

* [Single-responsibility Principle]({{< ref "#single-responsibility-principle" >}})
* [Open-closed Principle]({{< ref "#open-closed-principle" >}})
* [Liskov substitution principle]({{< ref "#liskov-substitution-principle" >}})
* [Interface segregation principle]({{< ref "#interface-segregation-principle" >}})
* [Dependency inversion principle]({{< ref "#dependency-inversion-principle" >}})

## Single-responsibility Principle

**A function or class should only ever have a single reason to change.**

It should have a single responsibility. If the description of a class or function requires the word and, it is a hint that that class or function may be doing too many things.

Which of the following code samples is easier to understand?

def generate_report():
    files = []
    for path in os.listdir("/nfs/site/disks/grid_storage/grid/xml_data/"):
        if os.path.isdir(path):
            for path2 in os.listdir(path2):
                if os.path.splitext(path2)[1] == ".xml.gz":
                    files.append(path2)
    job_count = 0
    for path in files:
        with gzip.open(path, "r") as f:
            tree = etree.parse(f)
        job_count += len(tree.getroot().findall("Job"))
    print(f"Total jobs: {job_count}")
def generate_report():
    job_count = 0
    for path in get_xml_files():
        job_count += len(find_jobs(path))
    print(f"Total jobs: {job_count}")

def find_jobs(path):
    root = get_xml_root(path)
    return root.findall("Job")

def get_xml_root(path):
    with gzip.open(path, "r") as f:
        tree = etree.parse(f)
    return tree.getroot()

def get_xml_files():
    for path in os.listdir("/nfs/site/disks/grid_storage/grid/xml_data/"):
        if os.path.isdir(path):
            for path2 in os.listdir(path2):
                yield path2
Even though the code in the second example is longer, it is easier to keep in your mind. Each piece does a single job, and is encapsulated such that the layer above it doesn’t need to know the details of how to locate XML files, parse them, or even necessarily how to find the jobs within.

See also: “Single-responsibility principle” on Wikipedia


## Open-closed principle

**Classes should be open for extension but closed for modification.**

## Liskov substitution principle

**The user of an interface should never need to know the implementation behind it.**

## Interface segregation principle

**Do not force users to depend on interfaces they do not need.**

Use the Reader and Writer examples from Go. Forcing the user to use a TextIO or while or whatever when it really only needs an Iterable[str] is bad design.

## Dependency inversion principle

**Depend on interfaces, not concrete implementations.**

TODO: library class which requires a MySqlDatabaseConnection parameter,
but then later wants to switch to a SqliteDatabaseConnection and can’t.

