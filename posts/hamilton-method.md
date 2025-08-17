<!--
.. title: Hamilton's Method
.. slug: hamilton-method
.. date: 2025-08-16 16:02:43 UTC-07:00
.. tags: algorithm, math
.. category: 
.. link: 
.. description: 
.. type: text
-->

Totally—here’s what the Largest Remainder (Hamilton) method is doing in our break allocator, and why it’s a good fit.

### What problem are we solving?

We have a fixed break budget (in 30-minute units) and k = (number of meetings − 1) break slots (one after each meeting except the last). We want to split that budget proportionally to the “weight” of each slot (the weight = the preceding meeting’s duration), while keeping:

total assigned = total budget,

longer meeting ⇒ longer (or equal) break,

:00/:30 alignment (so we only deal in multiples of 30 minutes).

### Hamilton method in a nutshell

Given:

integer total units (break budget in 30-min chunks),

positive weights w[i] (here: hours of the meeting before break i), i = 0..k-1.

Steps:

Compute ideal shares (as real numbers):
```
ideal[i] = units * w[i] / sumW
```

Floor each ideal:
```
base[i] = floor(ideal[i])
```

Count remainder units to distribute:
```
remaining = units - sum(base[i])
```

Sort slots by descending fractional parts of ideal[i] (i.e., ideal[i] - base[i]).
(Tie-breakers we use: larger weight first, then earlier index.)

Give one extra unit to the top remaining slots from that order:
```
base[idx_j]++ for j = 0..remaining-1
```

Convert units → minutes: break[i] = base[i] * 30.

This guarantees:

- The total exactly matches the budget,

- Each slot’s share is as close as possible to the proportional ideal (in an L1 sense, given unit granularity),

- No fractional 10-minute weirdness; everything aligns to :00/:30.

**Code snippet**: [https://github.com/leadingyu/small_tools/blob/main/OOD/schedule_meeting/DynamicMeetingScheduler.java](https://github.com/leadingyu/small_tools/blob/main/OOD/schedule_meeting/DynamicMeetingScheduler.java)

### Why it’s good here

Proportional fairness: longer meetings tend to get more break time.

Granularity-aware: we must use 30-min blocks; Hamilton rounds fairly.

Deterministic & simple: O(k log k) due to one sort on at most 6 slots (since workday is small).
