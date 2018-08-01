---
title: Use iostat -En instead of format
slug: use-iostat-ee-instead-of-format
date_published: 2015-01-21T05:49:18.601Z
date_updated:   2015-01-22T16:06:45.349Z
---

I generally see recommendations to use `format` to list the disks installed in a Solaris-like system. I find `format` rather annoying and consider this terrible advice.

I find `iostat -Een` to be superior in every way:

- Output can be filtered and sorted.
- The tool exits cleanly without interaction.
- The details are fuller.

I can think of only one drawback:

- `iostat -Een` is more difficult to remember than `format`.


