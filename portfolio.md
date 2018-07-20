---
layout: page
title : Portfolio
link : Portfolio
header : Portfolio
group: navigation
listed: false
width: 10
---

The following is a selection of my best work. It includes essays available elsewhere on this site, video recordings of recent presentations and links to more information on projects that I'm involved in.

## Writing

Math Envy and CoffeeScript's Foibles: [Part1](/2012/11/27/math-envy-and-coffeescripts-foibles/), [Part 2](/2013/01/09/math-envy-and-coffeescripts-foibles-2/) - CoffeeScript is a popular JavaScript replacement well know for it's syntactic flexibility, but the language's design has produced confusing interactions between terms. These posts formalize a subset of the language to reproduce one such issue, formalize semantic ambiguity using operational semantics, and then finish with a proposal for a tool to prevent similar issues in the future.

[Faster JavaScript with Category Theory](/2012/02/09/faster-javascript-through-category-theory/) - This post covers my initial work on formalizing the relationship between vanilla DOM manipulation functions and jQuery methods using Category Theory. It forms the basis for one talk and two essays.

[Splitting jQuery in Two, A Proposal](/2012/07/19/splitting-jquery-in-two-a-proposal/) - The relationship between the categories <b>Html</b> and <b>Jqry</b> highlighted a logical division of responsibility for jQuery methods. The result is better performance and a more modular architecture for the library.

[A Natural Transformation in JavaScript](/2012/03/22/a-natural-transformation-in-javascript/) - A natural transformation was a logical progression after constructing two categories and a functor in the previous writing on JavaScript and Category Theory.

## Open Source

[Vagrant](http://vagrantup.com) - Vagrant is a tool to make virtualizing development environments entirely painless. This has many benefits like deployment environment parity, fast developer on-boarding, and easy deployment testing. My involvement has been limited recently but I am the co-creator of this project.

[jQuery Mobile](http://jquerymobile.com) - jQuery Mobile is a UI kit and collection of browser fixes that make developing cross browser mobile web applications easy. I am currently employed by Adobe Systems as a development lead on this project.

[Wield](https://github.com/johnbender/wield) - Wield is the realization of the proposal made in [Splitting jQuery in Two, A Proposal](/2012/07/19/splitting-jquery-in-two-a-proposal/). It is a collection of very simple DOM manipulation methods that can be used at the core of jQuery's DOM manipulation methods. The primary project goals are modularity, compact size, and minimal JavaScript overhead for DOM manipulation.

## Talks

[Math Envy and CoffeeScript's Foibles](/2013/07/19/presentation-math-envy-and-coffeescripts-foibles/) - CoffeeScript is an extremely dynamic and flexible language that compiles to JavaScript. This talk takes one confusing term interaction and shows how it can be used to generalize about language construction.

<video x-webkit-airplay="allow" src="http://wpc.0B0C.edgecastcdn.net/000B0C/carsons/events/2013/FILive2013/d3-r309-230pm-JB.mp4" controls width="560px"></video>

[Faster JavaScript with Category Theory](/2013/08/29/presentation-faster-javascript-through-category-theory/) - After refining the results in three essays I created a thirty minute presentation around my thoughts on Category Theory and JavaScript. I gave the talk most recently at JQCONUK in Oxford to approximately 600 developers.

<iframe width="560" height="315" src="http://player.vimeo.com/video/71132093" frameborder="0" allowfullscreen></iframe>

[Progressive Enhancement on the Mobile Web](http://www.infoq.com/presentations/Mobile-Web-Development) - The mobile web is populated by an incredibly diverse set of browsers with wide ranging capabilities. In this light hearted talk I discuss some of the more difficult and subtle issues confronting developers working on the mobile web and present some solutions used in jQuery Mobile.

[Middleware as General Purpose Abstraction](/2012/04/28/middleware-as-a-general-purpose-abstraction/) - When Mitchell Hashimoto and I created Vagrant we eventually used a modified version of the middleware described in [PEP333](http://www.python.org/dev/peps/pep-0333/) to compose actions over virtual machines. For more information on Vagrant itself see the section on Open Source.

<iframe width="560" height="315" src="http://www.youtube.com/embed/fcNaiP5tea0" frameborder="0" allowfullscreen></iframe>
