---
layout: post
title: Contributing to jQuery Mobile
tags:
- javascript
- jquery
- vagrant
status: publish
type: post
published: true
meta:
  _edit_last: "1"
  _wp_old_slug: ""
---

<strong>UPDATE</strong>: I've updated the vagrant branch used below to support the newest version of Vagrant.

This is more substantial version of the lightning talk I gave at LA RubyConf last Saturday. If you're a <a href="http://vagrantup.com">Vagrant</a> user already, this will be familiar. If not, don't worry, you're about to get a quick introduction to its power as a tool for web development. If you have any questions feel free to comment or find me on twitter <a href="http://twitter.com/johnbender">@johnbender</a>.

<h2>Vagrant</h2>

jQuery Mobile is set up to use php for serving the docs and tests, both of which you will need in order to start hacking on the library. Instead of installing apache and mod_php on your workstation though, we're going to build a vm using Vagrant that will serve it up.

First you'll need to install VirtualBox and Vagrant which is covered in glorious detail by Mitchell's <a href="http://vagrantup.com/docs/getting-started/index.html">documentation</a>. If you're questioning the time investment at this point I can promise you that its worth it. Vagrant will change the way you look at web development.

<h2>Clone and build</h2>

The next step is to fork and clone the jQuery Mobile repo

<pre>
$ git clone git@github.com:yourgithandle/jquery-mobile.git
$ cd jquery-mobile
</pre>

add my fork as a remote

<pre>
$ git remote add geewilikers https://github.com/johnbender/jquery-mobile.git
</pre>

track the branch with the Vagrant configuration

<pre>
$ git fetch geewilikers
$ git branch --track vagrant geewilikers/vagrant
$ git checkout vagrant
</pre>

make sure to add a base box from which to build our JQM virtual machine

<pre>
$ vagrant box add base http://files.vagrantup.com/lucid32.box
</pre>

and then fire that sucker up!

<pre>
$ vagrant up
</pre>

About 2 to 5 minutes later, depending on your connection speed, you should be able to hit the <a href="http://33.33.33.10/">docs site</a> served by your brand new vm. You can also view some <a href="http://33.33.33.10/tests/unit/core/">tests</a>.

<h2>Making changes</h2>

So now that the site is running how do you make changes? What do you do with this new branch? What does the workflow look like?

Its actually quite simple. Vagrant lifts the project directory into the virtual machine as a "shared folder" ( it actually mounts the folder with NFS because VirtualBox's shared folders have extremely poor performance for complex file structures ). This means you can edit any files you like locally and the changes will be represented immediately in the vm. Just make sure that you checkout master first and use the vagrant branch any time you want to run a vagrant command.

For example, if you wanted to fix a typo in the docs and had just finished building the vm:

<pre>
$ git branch
  master
* vagrant
$ git checkout master
... hack hack ...
$ git add .
$ git commit -m "changed the docs to be more awesome"
</pre>

There's no need to stay on the vagrant branch, its only necessary when you want to interact with vagrant. For example if you wanted to suspend the vm after finishing some work:

<pre>
$ git checkout vagrant
$ vagrant suspend
</pre>

When you want to start working again, just resume the suspended vm:

<pre>
$ vagrant resume
$ git checkout master
</pre>

And that pretty much covers the workflow. If you ever want to go rooting around in the vm itself, which in this case I find to be a rare occurrence, you can simply ssh in:

<pre>
$ vagrant ssh
</pre>

In the worst case where your vm just doesn't seem to be working at all, just destroy it and rebuild it from scratch.

<pre>
$ vagrant destroy
$ vagrant up
</pre>

<h2>Pull requests</h2>

Now that its dead easy to make changes to JQM and you're ready to dive in and help out, where's the best place to start?

As of this writing the team is focusing on bug fixes and testing. Feature requests are being moved to a backlog for post beta/1.0 consideration as we focus on platform support and stability. That means if you want to contribute your best bet is to find an unclaimed issue ( ie one that isn't tagged with a person's name ), in the <a href="https://github.com/jquery/jquery-mobile/issues">issues list</a> on github and try your hand at a fix.

Once you're happy with your solution submit a pull request and ... wait. We're doing our best to get to all the pull requests but we get a fair amount and you might have to wait a couple days to get a response while we sort through them.

To speed the process along there are a couple important things you can do:

<strong>1. Rebase!</strong>

If/When your pull request branch falls behind master don't merge - rebase! By rebasing and resolving the conflicts you make the application of your commits to the existing master much easier for the reviewer.

First thing you'll want to do is add the original git repo

<pre>
$ git remote add jqm git://github.com/jquery/jquery-mobile.git
</pre>

Then you can fetch the latest from the original jqm and rebase your changes on top of it

<pre>
$ git fetch jqm
$ git branch
* master
  vagrant
$ git rebase jqm/master
... resolve conflicts ...
</pre>

<strong>2. Separate white space/refactor commits</strong>

If you want to add in a quick refactor, like renaming a variable, or cleaning up some white space keep it in a seperate commit from the bug fix/feature. That way the reviewer doesn't have to fight to separate the important changes from the rest.

<strong>3. Test</strong>

This really cannot be over emphasized. If you fix a bug please add a test for it. In this way we can guarantee that there aren't any regressions in the future as the project makes changes and improvements. Just take a gander at the other tests to see what they should look like and dive right in. Even better, if you want to learn the library, just start writing tests for whats already there, you'll be making a huge contribution to the stability of the code base.

<h2>Contributing to other projects</h2>

The most important part of all, which I didn't get enough time to talk about at the conference, is that you don't have to be a rocket scientist or hacker extraordinaire to contribute in a big way to an open source project you're interested in. How do I know this, you may ask? Because I'm neither one of those things.

Sadly, I spent a lot of time on the outside looking in, thinking "I'd just be making trouble" or "I doubt I'd have any meaningful impact even if they did accept my submission". If you tend to think on the same level, stop. Go fix a bug. Go write a test. Don't let the fear of someone rejecting your submission stop you from trying in the first place.

Even if it doesn't get accpeted, take the feedback to heart, get better, and try again. Good projects will have a polite and helpful response for you if they don't think your fix is quite ready to be merged in. As someone who reviews pull requests I know that the most important asset the project has is its contributors and users. I see it as my most important job to make sure they know we appreciate all their hard work. ( credit to Yehuda Katz's <a href="http://www.infoq.com/presentations/Open-Source-Project-Like-Rails">talk</a>, and many discussions with Mitchell Hashimoto for those ideals).

<h2>Dive right in</h2>

Hopefully if you were on the fence about contributing to JQM or any other oss project this was the nudge you needed to give it a try. Open source is dead if people don't jump in to contribute and make it better, so I hope to see your pull request in our queue sometime soon!
