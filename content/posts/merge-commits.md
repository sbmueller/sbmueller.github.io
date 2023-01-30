---
title: "The Problem with Merge Commits"
date: 2023-01-30T16:28:09+02:00
tags: ["opinion"]
draft: false
---

When working with git professionally over some years, a question raises up
quite often: Which merge strategy to use? I don't want to write a comprehensive
guide about merge strategies, since this has been addressed
[often](https://www.atlassian.com/git/tutorials/using-branches/merge-strategy)
already. Instead, I want to point out one argument against merge commits that
is important to safety relevant software in particular.

To explain the problem in detail, I created an [example
project](https://github.com/sbmueller/the-problem-with-merge-commits) in my
GitHub profile along with an explanatory readme, so I won't duplicate the
explanation here again. In short, merge commits could be dangerous since they
create unreviewed and untested code states.

There are many arguments for different merge strategies and while always
dependent on the context, I'm a believer in fast forward merges since many
years. I never faced version control issues, maintained a clean linear history
and was always forced to resolve issues before merging. However, I understand
in projects with a high merge rate, it might not be feasible to rebase and
adapt, since in the meantime new changes might be added to the target branch.
