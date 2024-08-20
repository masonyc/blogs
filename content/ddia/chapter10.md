---
id: chapter10
aliases: []
tags: []
date created: 2024-08-20T08:00
date modified: 2024-08-20T08:00
---

# Batch Processing with Unix Tool

Unix **sort** command: chunks of data can be sorted in memory and written out to disk as segment files, and then multiple sorted segments can be merged into a larger sorted file. Mergesort has sequential access patterns that perform well on disks.

The sort utility in GNU Coreutils (Linux) automatically handles larger-than-memory datasets by spilling to disk, and automatically parallelizes sorting across multiple CPU cores. This means that the simple chain of Unix commands we saw earlier easily scales to large datasets, without running out of memory. The bottleneck is likely to be the rate at which the input file can be read from disk.”

## The Unix Philosophy
