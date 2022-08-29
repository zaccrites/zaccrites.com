---
title: DevOps Explained
date: 2022-08-22
draft: true
tags: [devops]
# slug: magical-world-of-sat-solvers-part-2
---


## What is CI/CD?

CI/CD is a process which enables developers to deliver features to customers faster, more easily, and with greater quality, while also reducing development complexity and getting feedback back to developers more quickly.

## What is CI?

Continuous integration is a technique for managing development complexity by encouraging developers to deliver their changes to the master branch as often as possible. The longer a feature branch goes on, the more likely it is that it will introduce integration problems when it is delivered. Smaller iteration cycles avoid this problem by forcing developers to make small changes incrementally, and small changes are easier to integrate, test, and deploy. This also aligns with Agile software principles, which prefer smaller user stories with limited scope to large ones which go on for days or weeks and make many changes.

To aid in this effort, build automation (frequently called a “CI pipeline”) is ready to lint, build, test, and package the software (or hardware) product automatically. This gives developers fast feedback on problems with their changes, frees reviewers from manually flagging these issues and verifying them, and ensures that code merged to the master branch has good quality, is fully tested, and is packaged up and ready-to-“ship” at all times.

## What is CD?
This concept of “always-ready-to-ship” enables continuous delivery, whereby merging changes to the master branch not only delivers a source code update but also triggers a deployment to production. In addition to build automation, continuous delivery emphasizes deployment automation to encourage operations personnel to deploy as often as possible (even multiple times per day). This delivers valuable features to customers quickly and often, and delivers immediate feedback to developers.
