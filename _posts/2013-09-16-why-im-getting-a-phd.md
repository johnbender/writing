---
layout: post
title: "Why I'm Getting a PhD"
tags:
- career
- education
status: published
type: post
published: true
listed: true
vote: http://news.ycombinator.com/
---

The 26th is the first day of instruction in the first academic year of my PhD in Computer Science at UCLA. I have a two year old daughter, an incredibly supportive wife, and I just turned 30.

Unsurprisingly, I've been asked many times why I want to get a PhD and I rarely get a chance to explain my thinking fully. This post is both a detailed account of my reasons and a counterpoint to the popular opinion that there isn't much value in higher education in Computer Science.

## Why Bother?

Any confusion or surprise over my decision to go back to school can generally be reduced to a simple idea: the success I would otherwise have while getting my PhD is of greater absolute value than the education.

This comes in many reasonable forms. Why are you leaving such a high paying job? Will you be able to speak at conferences? When will you find time to contribute to open source? Couldn't you just work on those ideas in your free time? These questions are asked directly and without implication.

In some cases there is a subtle suggestion that academia and higher education are losing their value. Why would someone pursue a PhD when there are people without any higher education earning comparable salaries and working on important projects? I never hear this asked explicitly, but it comes through in many conversations and the idea that education isn't important is pervasive in the software "meritocracy".

In each case, I'm consistently left wanting more time to describe my thinking and the personal experiences that landed me in a PhD program.

## Career Path

When I got my first legitimate management position as the director of engineering at a consultancy/incubator it quickly became clear that management wasn't for me. I enjoyed seeing my teammates succeed. I enjoyed building and refining Process. I enjoyed winning business. I enjoyed the subtleties of communicating complex technical ideas to people with all types of backgrounds and experience. By my own estimation I was even pretty good at all those things.

The thing that got to me was the amount of time I spent staring at my email client. After a year I missed the technical aspects of the job, and I started to think about my long-term career options. At the time I saw three paths that didn't route back through school.

1. **Avoid titles and stick with engineering** This was, at least initially, the path I took. I left for Adobe and a position working full time on jQuery Mobile.
2. **Grab the title and climb the ladder** Management positions pay a lot of money, and there's a lot of room for advancement. Not all require you to hand over your keyboard.
3. **Employee Number One or Co-Founder** First employee/Co-Founder positions are available for experienced "generalists" as far as I can tell. This is purely based on personal observations and conversation with friends who are better informed on the topic.

Of course, there are a lot of subtle variants to each of these, but this was my assessment. I can say with confidence that my move to Adobe put me in an ideal engineering position. Working on an OSS project full time gave me the freedom to pursue long term solutions to difficult problems and I still had access to the good parts of a large corporate support infrastructure.

Unfortunately, the problems I gravitate towards are not normally assigned to engineers and attempts to marry my interests with my domain of expertise [1,2] have met with understandably tepid responses. The ideas are hard to follow in short presentations, most people don't have time to read marathon blog posts, and the underlying work isn't funded by my employer. All of that makes perfect sense but it means I can't focus on the problems that I'm interested in.

## What Interests Me

The things that keep me up past midnight working and learning are technical. Problems both large and small that remain unsolved or problems where the solution seems unsatisfactory to me. A short and incomplete list in no particular order:

* Descriptive declarative languages that aren't
* Correctness in dynamic languages
* System configuration and state management
* Adoption, understanding, and value of advanced type systems
* Direct applications of mathematics to programming languages
* Semantics based approaches to hot code loading
* Preventing language bugs/quirks during language creation

A few of these might align with a job posting somewhere but most don't. More importantly, I'm not as interested in the implementation as I am in conceiving a solution and understanding its value. I'm sure that will sound like laziness to some, but the part I enjoy the most is thinking through the idea. Most of the time an implementation is an important part of that, and I enjoy hacking as much as anyone, but sometimes it's not. Sometimes a more abstract representation of the problem is the best way to predict the relative value of a solution before committing time and effort to an implementation. Again, this isn't the type of work that engineers get funded to do.

## On Academia

Perception of academia varies depending on the field and an individual's cultural disposition. In the software community there is a certain sense of respect for the research and science on which our careers are built. People have even sort of deified greats like Alan Kay, Alonzo Church, and Alan Turing.

More and more though, the word "academic" is being used as a pejorative to mean irrelevant, unimportant, or a wasted effort. It's easy to wave this away as an issue of sloppy semantics but I think it highlights the suffering perception of academics. In my experience working on research projects and doing a lot of reading over the last few years, much of the stigma appears to result from two issues. The first is, what people think research should be for, and the second is the occasionally impenetrable nature of formalism.

Computer Science research is frequently accompanied by proof of concept software. It might be poorly constructed, hard to get working, or even hard to find. In turn, that can lead to a poor opinion of the research and researchers, but the implementations are rarely the primary contribution of the work. The goal of the researcher is *not* to provide implementations or information directly to industry, but rather to produce a solution to an outstanding problem. The work of translating that solution into something "well built" or even concrete is frequently left to the reader.

Unfortunately, reading Computer Science papers can be hard work. The deluge of notation can be discouraging. It can even seem like unnecessary ceremony, but a lot of ground has to be covered in the limited space provided for conference publications. Moreover, formalism and logic are the most important tools we have when streamlining and finding consensus on a good solution. In a perfect world all the necessary context would be easily accessible, and the formal tools used to establish properties of a solution would be easy to understand. Sadly that's not the case, but it doesn't diminish the value of the "encoded" contribution.

It's *because* of these things and not in spite of them that I'm going back to school. I want to work in an environment that explicitly promotes a focus on the fundamentals of a problem and requires that the utmost care be taken when presenting a solution.

## The Fourth Path

I guess you could say I'm taking The Fourth Path™.

While I may be new to the academic environment, I have taken efforts to ensure that my impressions are not naive. I have been working on research projects since last year with folks at UCLA and I have been attending regular reading groups. I also know what problems I want to work on; one of which I'm pursuing with a strong blessing from my adviser.

Even if this doesn't turn out as planned, at least I know that the "what" and "how" of my work will fit and that's why I'm getting a PhD.

<br/>

### Acknowledgments

Thanks to [@keyist](https://twitter.com/keyist) and [@melliebe](https://twitter.com/melliebe) for their notes and revisions.

Special thanks to [Professor Philip M. Johnson](
http://philipmjohnson.wordpress.com/) who has helped me continuously since I graduated from the University of Hawaii more than seven years ago. His advice has been extremely important in my long and winding path back to academia.

### Footnotes

1. [Presentation: Faster JavaScript through Category Theory](/2013/08/29/presentation-faster-javascript-through-category-theory/)
2. [Presentation: Math Envy and CoffeeScript's Foibles](/2013/07/19/presentation-math-envy-and-coffeescripts-foibles/)
3. "Less" is not, "not at all"

I also have a short list of things I'm *not* interested in. I intended to include this in the main body of the post but it seemed like a distraction.

As a web developer and someone who's worked almost exclusively on the client side for the last few years I have a fairly long list of things that do not interest me or are an active source of frustration. Nearly all of them would be impossible to describe as "general" problems.

* Esoteric browser bugs (yes they still exist in abundance)
* The culture/necessity of JavaScript micro-libraries
* Effort required to provide broad access to web based content
* Dynamic programming languages as an industry default
* Non-transferable esoterica (e.g., how iptables works on CentOS, Chef's attribute resolution order)
* Large JavaScript projects

I'm not above working on, or around, these problems. For a web developer many of these are simply facts of life. I'm just not content to let them bother me indefinitely which means contributing to solutions or moving on.
