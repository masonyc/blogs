---
id: chapter9
aliases: []
tags: []
date created: 2024-08-08T08:00
date modified: 2024-08-08T08:00
---

# Introduction

If two nodes both believe that they are the leader, that situation is called split brain, and it often leads to data loss

# Linearizability (Atomic consistency / Strong Consistency)

the basic idea is to make a system appear as if there were only one copy of the data, and all operations on it are atomic

In a linearizable system, as soon as one client successfully completes a write, all clients reading from the database must be able to see the value just written. Maintaining the illusion of a single copy of the data means guaranteeing that the value read is the most recent, up-to-date value, and doesn’t come from a stale cache or replica
