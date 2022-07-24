---
title: "How to report software issues"
date: 2018-12-16T12:30:32+01:00
tags: ["opinion"]
draft: false
---

## TL;DR

A good report of a software issue should look like this:

- What is the initial position of your system? What operating system are you
  on? What versions of your OS and of the software under question were used? Is
  there any other software like previous versions that might affect behaviour?
- What exactly did you do? How did you install the software? Which dependencies
  did you install and how did you do it? How were you trying to use the
  software? Be specific! What commands did you execute? What files did you use?
- What did you expect to happen? Why did you think the former?
- What happened instead? Provide all output the software is providing.

## Why is this important?

As a maintainer of [a software](https://github.com/gnuradio/gr-inspector), I
sometimes receive issues reported by users. However, most users fail in their
first attempt to provide a good issue report. A typical query I experience
might look like this

> Dear Sir,
>
> I tried to use your software but it did not work. I did the commands in the
> readme, but there were errors. Please help!

I think it should become pretty clear that this query does not give me any
possibility to provide help. Try to put yourself in my place: How would you
help this user? I need to reproduce the issue to understand what is going wrong
and to fix it. In other words: I need the user's help to help them!

With complex software and operating systems these days, there are a trillion
things that could go wrong when trying to use a software. In order to provide
help and narrow down the issue, users should provide as much information as
possible. Please keep in mind the proposed structure at the beginning of this
article if you happen to seek help on a software that you are trying to use.

