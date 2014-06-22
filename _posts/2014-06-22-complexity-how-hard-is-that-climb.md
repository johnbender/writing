---
layout: post
title: "NP-completeness: how hard is that climb?"
tags:
- math
status: draft
type: post
published: true
listed: false
---

turing machine
robustness of turing machine

difficulty of problems
- easy vs hard
- efficient vs inneficient
- should I bother trying to come up with something clever?
- polytime vs "not polytime"

definitions
- polynomial time
- non-determinstic polynomial time
- certs vs non-determinism

np-completeness
- two components
- "at least as hard" as other problems that are in NP
- also in NP

mountain analogy
- P = hours, day trip, supplies: water
- NP = multiple weeks, supplies: other people, oxygen, food, shelter, wifi
- reduction = bridge
- P-reduction = bridge that takes at most a day to cross
- friend asks you to plan a climb of mountain X
- you don't know anything about mountain X
- you know how hard mountain Y is
- distinction between really hard mountains and regular mountains
- just height?
- regular mountains doable in a day
- hard mountains, many days, really dangerous, above 8000m
- you can convincingly show that X is similar to mountain Y

polytime reduction, bridge
- build a  bridge horizontal or higher, arrive ON the moutain, not on TOP (polytime reduction, hardness)
- the bridge can only take regular mountain time (1 dayish) to cross
- otherwise the bridge will ruin your comparison
- if it takes the same amount of time to cross the bridge as it would to climb a huge moutain you may have climbed a huge mountain in the process!

alternately
- obscurred by clouds
- you set yourself up so you can measure that it's at least as tall as mountain Y under the clouds
- another way to put it: if you were to build a horizontal bridge between the top of mountain Y, then climb mountain X up to the level of the bridge, cross the bridge,  you would end up at the top of mountain Y. That means X is at least as hard (height wise) as mountain Y
- if it takes the same amount of time to cross the bridge as it would to climb a huge moutain you may have climbed a huge mountain in the process!
- this is the polynomial time reduction, if you can climb mountain X in a few days, then you can certainly climb mountain Y in a few days if only by climbing X to the bridge and then taking a day to cross.
- seems backwards, but the thing we're trying to prevent is classifying some mountain Z as being as hard as Y when it's not. So if you couldn't cross that hypothetical bridge from Z to Y and be at the top then Z is not the same as Y. Eg, you can't climb a grassy gnoll and then cross a horizontal bridge in a day and be at the top of moutain that takes a week to climb.

- can't build a day trip bridge down from a week long hike mountain to a day trip mountain, the mountain is too tall, the brige won't reach
- could be possible, teleportation? We don't know!
- if you could find a way to bridge from a day mountain to a weeks mountain then you could just hop other bridges from that weeks long mountain to any other weeks long mountain in days which would make all weeks long mountain trips take only days instead of weeks!
- similarly if we could show that there is a reduction from any NP-complete problem to a problem in P then we could use polytime reductions from that NP-complete problem to any other to solve it in polynomial time.

In the hard mountains class
- knowing the mountain is at least as hard as something else has value
- you could call the trip off based just on that
- ideally show that it's just as hard and not harder
- eg, k2 and not olympus mons
- satellite photos etc

notes
- days vs weeks gulf doesn't capture the gulf between P and NP but serves here
