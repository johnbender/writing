---
layout: post
title: "NP-completeness: How hard is that climb?"
tags:
- math
status: draft
type: post
published: true
listed: false
---

difficulty of problems
- easy vs hard
- efficient vs inefficient
- Assuming we're looking for a general solution should I bother trying to come up with some clever way of conquering the problem or are we stuck trying all the different possibilities?

## Climbing a Mountain

Imagine that a friend came to you and said that she would like you plan a trip up an obscure mountain,  **X**. Obviously you need to know how difficult the trip is going to be to do a good job. If it's only a quick day hike to the top you probably just need some sandwiches and water, but if it's a multi-week expedition then you'll need a more comprehensive set of supplies.

Further, imagine that you are unable to find any information about the difficulty of climbing up mountain **X**! Your intuitive reaction would probably be to find *comparable* mountains to get a sense for the difficulty of mountain **X** using some sort of comparison. Fortunately, right near by there is a mountain Y that is well known. In fact it's close enough that a comparison by height (which we'll use as our proxy for difficulty) is possible save for that fact that peak of mountain **X** is constantly covered by clouds.

Now, all we really know about mountains X and Y is that X is taller and therefor more difficult than Y. We need to get some sense for how difficult mountain Y is so that we can then infer something useful about mountain X. Since we're using vision as our measuring device our system of classification is going to be fairly coarse. That is, if we're positive a mountain can be scaled in a day (think, large hill) then it's probably visually pretty small and if it's going to be on the order of a few weeks then it probably looks pretty serious (think, K2).

Lets call these two classes of mountains D and WD, for "Day" and "Weeks and Days". When a mountain will take at least a day to hike we say it's D-hard. When it will take at the least weeks and days to hike then we say it's WD-hard. So if mountain Y is D-hard then clearly mountain X is D-hard and similarly so if Y is WD-hard.

Lets assume that Y is "Weeks and Days"-hard, and that means we know that X is too because it's at least as big as Y. But there's still a problem, we don't know *exactly* how big mountain X is because the top is obscured by clouds. We only know that it's as hard as Y to climb. If X ends up being as tall as Olympus Mons we should probably just forget it. So, you wait for a very clear day when the top of mountain X is visible and comparable to the top of mountain Y and verify that mountain X is of roughly similar size to mountain Y! Now we say that mountain X is WD-complete. That is, we know it's at least as hard as another "Weeks and Days" mountain and we feel confident that it will *only* take weeks and days and not, for instance, months and years.

## P, NP, NP-hard, NP-complete

You may not be a climbing fan but this serves to illustrate the basic process of establishing when a problem is NP-complete. To make the analogy clear we'll define, P, NP, NP-hard and NP-complete and link them to climbing.

## Polynomial-time Reduction

We used vision to verify the difficult of mountains but with the complexity of decision problems we'll need some math and the complexity of our device for measurement plays a critical role in it's validity.



- P = hours, day trip, supplies: water
- NP = multiple weeks, supplies: other people, oxygen, food, shelter, wifi
- reduction = bridge
- P-reduction = bridge that takes at most a day to cross
- friend asks you to plan a climb of mountain **X**
- you don't know anything about mountain **X**
- you know how hard mountain Y is


definitions
- polynomial time, there exists a turing machine that can decide the inclusion
- non-determinstic polynomial time
- certifiable in polynomial time, there exists a turing machine that can verify a solution for the inclusion in
- containment diagram
- problems in NP are also in P
- that is any, problem in P has a turing machine that can also act as a verifier by ignoring the cert.
- so it's not super useful to just say that a problem is in NP (because that means it could be in P) we want to know whether we think it can be solved in polynomial time.

## Decision Problems

Problems like [Traveling Salesman](http://en.wikipedia.org/wiki/Travelling_salesman_problem) are often characterized in their optimization form, in this case "find the shortest possible route that visits all cities". The decision version is whether there exists a path of some predefined length that visits all the cities. Obviously if you can solve the first you know the answer to second but the key difference is that, in the second case the output is a simple yes or no answer. Here we'll concern ourselves with decision problems only [footnote on why].


np-completeness
- two components
- "at least as hard" as other problems that are in NP
- NP-hard means don't try to be clever if you're looking for a general solution
- also in NP

mountain analogy
- P = hours, day trip, supplies: water
- NP = multiple weeks, supplies: other people, oxygen, food, shelter, wifi
- reduction = bridge
- P-reduction = bridge that takes at most a day to cross
- friend asks you to plan a climb of mountain **X**
- you don't know anything about mountain **X**
- you know how hard mountain Y is

Lower bound on the difficult of **X**
- **X** is obscurred by clouds, the bottom of the clouds are the lower bound
- Set yourself up so you can measure that **X** is at least as tall as mountain Y under the clouds
- that is, if you were to build a horizontal bridge between the top of mountain Y, then climb mountain **X** up to the level of the bridge, cross the bridge,  you would end up at the top of mountain Y. That means **X** is at least as hard (height wise) as mountain Y.
- If Y is in NP then **X** is NP-hard

Bridge
- if it takes the same amount of time to cross the bridge as it would to climb a huge moutain you may have climbed a huge mountain in the process!
- this is the polynomial time reduction, if you can climb mountain **X** in a few days, then you can certainly climb mountain Y in a few days if only by climbing **X** to the bridge and then taking the bridge across
- seems backwards, but the thing we're trying to prevent is classifying some mountain Z as being as hard as Y when it's not. So if you couldn't cross that hypothetical bridge from Z to Y and be at the top then Z is not the same as Y. Eg, you can't climb a grassy gnoll and then take a day bridge and be at the top of moutain that takes a week to climb.

Restrictions
- so far as we know can't build a day trip bridge down from a week long hike mountain to a day trip mountain, the mountain is too tall, the brige won't reach
- could be possible, teleportation? We don't know!
- if you could find a way to bridge from a day mountain to a weeks mountain then you could just hop other bridges from that weeks long mountain to any other weeks long mountain in days which would make all weeks long mountain trips take only days instead of weeks!
- similarly if we could show that there is a reduction from any NP-complete problem to a problem in P then we could use polytime reductions from that NP-complete problem to any other to solve it in polynomial time.


- knowing the mountain is at least as hard as something else has value
- you could call the trip off based just on that
- ideally show that it's just as hard and not harder
- eg, k2 and not olympus mons
- satellite photos etc

notes
- days vs weeks gulf doesn't capture the gulf between P and NP but serves here

notes on things being omitted for simplicity
 - most (all?) optimization problems that are NP-hard are not in NP since verifying that a solution is optimal is at least as hard as finding the solution itself!
 -
