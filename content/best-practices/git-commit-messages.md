+++
categories = ["recipes"]
tags = ["practices", "spring boot", "microservice", "application development", "git"]
title = "2.Git Commit Best Practices"
description = "Git Commit Best Practices"
date = 2021-01-05
weight = 10
draft = false
authors = ["Rohan Mukesh"]
+++

## CONTEXT
This recipe provides details of how to write _good commit message_.

## What is a commit message?

The `commit` command is used to save changes to a local repository after staging in Git. 
However, before you can save changes in Git, you have to tell Git which changes you want to save. A great way to do that is by adding a commit message to identify your changes.
   
## Why commit messages are important?

1. Commit messages provides a way to communicate with your developers on a project. 
   The change defines how you achieved something, but the _commit message_ explains _why_ you are doing it. 
   It should provide enough context to avoid a developer wonder why code is written like that.

1. Commit messages are not only for coworkers but also for our future self as Peter Hutterer explains in this [article](http://who-t.blogspot.com/2009/12/on-commit-messages.html)

> Any software project is a collaborative project. It has at least two developers, the original developer and the original developer a few weeks or months later when the train of thought has long left the station.

1. Git commands like `log`, `rebase`, `cherry-pick` can provide great insights from commit history if the commits are meaningful. Writing a good commit messages makes it easier to review
   with `git log -p`

## How to write a good commit message

1. Chris Beams describes these three points in his [article](https://chris.beams.io/posts/git-commit/): How to Write a Git Commit Message.
> Style. Markup syntax, wrap margins, grammar, capitalization, punctuation. Spell these things out, remove the guesswork, and make it all as simple as possible. The end result will be a remarkably consistent log that’s not only a pleasure to read but that actually does get read on a regular basis.

> Content. What kind of information should the body of the commit message (if any) contain? What should it not contain?

> Metadata. How should issue tracking IDs, pull request numbers, etc. be referenced?

1. Here are 7 rules to write a good commit message:
   * Separate subject from body with a blank line.
   * Limit the subject line to 50 characters.
   * Capitalize the subject line.
   * Do not end the subject line with a period.
   * Use the imperative mood in the subject line.
   * Wrap the body at 72 characters.
   * Use the body to explain what and why vs. how.
    
