---
comments: true
date: 2010-03-30 15:30:56+00:00
layout: post
slug: camp-and-some-code
title: Camp and some Code
wordpress_id: 7
categories:
- Events
- Rants and Raves
tags:
- CodeCamp
- Event
- Rant
- Rave
published: true
comments: false
---

On Saturday the 27th of April I attended my first MS CodeCamp in Belgrade. Up to now I tried to keep up either by livecasts, or by later watching the recordings (if available).
<!--more-->
I had mixed expectations about it, on one hand the agenda promised some interesting topics, on the other hand I had a nagging suspicion of upcoming disappointment in the back of my head. What I got was a nasty case of [NIH](http://en.wikipedia.org/wiki/Not_Invented_Here) mixed with some trivial examples, dashed with some incompetence.

Let me be clear, this is a codecamp, no rockstars here,  just us poor <del>developers</del> drones. I understand that some people never had a chance to speak in public, but for gods sake, once you get that chance at least **make an effort**.

First off let me express my biggest complaints, the room used had two problems:

  1. the seats and their respected legroom were minuscule, they might fit your standard run-of-the-mill geek/nerd stereotype, you know, pimple faced, near sighted short skinny, slender frame irony spewing bastards, but (un)fortunately I did not turn out quite like that, I got 110kg of meat/muscle (and some fat that goes with that thank you), 187cm height (and have been told on several occasions that a rabid grizzly bear has a nicer personality than me). No way in hell I can fit nicely into those masochistic seats made for theater lovers as punishment for wanting to see some non-headlining play of god-knows-which alternative troop.

  2. this is a code camp, someone might actually bring a notebook <gasp/> and whats worse he might actually want to use it <gasp/>.

That being said, lets move along to sessions...

### Session 1 - Code Refactoring

When I saw this session I thought, great, someone will demonstrate techniques how/when/why to refactor code because you can never have enough of those talks, that made me completely unprepared for what was coming.
Session turned out to be a R# 101, which in it self would not be too bad if it weren't for a few snags.

First off the two guys that were speaking were a bit too shy to make an impression on someone, if they truly made an effort I deeply believe that they could have done a better job.

Second is the problem of the refactoring, it was presented as "ok, we all know why it needs to be done, and we all know how it should be done, now let me show you something that will make your life a lot easier...  Resharper".

Then they actually had the gaol to present the Resharper as the only solution out there for code refactoring, and that it was either R# or do it manually because Visual Studio has no refactoring tools.
Really? How about this quaint lil' tool called [CodeRush](http://devexpress.com/Products/Visual_Studio_Add-in/Coding_Assistance/) and it's supplement [Refactor Pro](http://devexpress.com/Products/Visual_Studio_Add-in/Refactoring/) both made by [DevExpress](http://devexpress.com/)?

Nevermind the fact that they were using R# 5.xx EAP or nightly build (which at that time was not declared RC and was bringing a lot of improvements compared to current 4.xx version)

Let me be clear here and say that I do not think one or the other is a superior/better tool, all I am saying is that there are alternatives out there.
And they can be crazy fun too, check out this video called [Guitars and Code with Mark and Mehul](http://tv.devexpress.com/#GuitarCodeInterview.movie) from last years PDC involving an XBox Guitar, CodeRush/Refactor Pro and Visual Studio.

The example project used was a one-off not-even-close-to-real-world-project type of thing, that had 10-12 classes, each one demonstrating each type of R# refactoring abilities. At first glance it is not that big of a problem, but as a R# user I know very well of the pain of working with R# on real world project, and the tedious VS freezing while the solution is being loaded, and R# parsing it.
Sometimes it becomes so bad that swift kick in the nuts is actually preferable to the waiting game.
Quite often the endgame result is the reset of the Visual Studio, which then proceeds to load the solution as it should, and like everything is rosey and the crash never happened.

As they were going through the session, file by file, trying to give a hands on demonstration of R# abilities the loudest sound in the room was their notebooks keyboard clicking, with very little talk and explanations, and quite often with error results followed by some funny sounds they made.
These guys use R# everyday (or so they say), they come to CodeCamp to give a presentation of it, and **DO NOT USE** R# powerfull code snippets to quickly dump in the code that is working and is prepared before, are you for real?

As the allotted time for the session passed, and Q&A time came up, it was cut down to none by the organizers with explanation that they broke through the time limit. In reality I think they tried to save them from getting their asses handed to them.

I for one wanted to know what optimizations in settings could be used to alleviate that painfull waiting time, I am not sure I would actually get a response on that. And by the looks of few people in the audience I was not the only one pissed off.

### Session 2 - New Features in ASP.Net MVC 2 (SectionAddViewModel)

This session was of great interest to me, only to be completely brought down by a lame attempts at humor, piss poor copy of the HaHaa show and a weird fetish for [Phil Haack](http://haacked.com/). I swear, if they did a Monty Python routine and messed it up, I would have gnawed my own leg off, and beat them senseless with it.

In a blur of crap they spun up, I can vaguely recall the explanations of what areas are, how to use the AcceptVerbs attribute , that the security guy can't spell if his life depended on it, and that the protection from xss is much better and easier in WebForms.

One thing I will give these guys is that they were frank and said they prepared the talk in a hurry (think couple of hours before the codecamp).

Hour of talk and poor stand up comedy comes down to three sentences, I think they could not have done much worse.

### Session 3 - Practical Agile

This session was very good, clear and concise. Small wonder considering that the speaker is a real speaker and an agile coach.
My feeble attempts of it's description would just diminish it.

### Session 4 - Business Application Architecture (or m stands for "much better than CSLA")

I am always eager to learn about architecture, and improve my knowledge in this area, but for the life of me I do not see the link between the Business Application Architecture and UI Framework code generator.

Basically the speaker created a nice drag'n'drop UI framework with namespace name Jmf, that when done, would generate a VS C# solution that can then be compiled and used.

Fine, I'll bite, and go along, so he is demonstrating how it's all wired, how the changes are propagated, and the usage of command pattern, I guess that this has some purpose and brings us one step closer to that millennium long dream of actually creating applications without developers, and now managers can cut out the malignant putrid tissue of developers, thus finally achieving the coveted simplicity and productivity.

At some point one of the audience members noted that his framework has a lot of similarities to [CSLA](http://lhotka.net/cslanet/), after that came a slew of features that after being shown were followed with a comment that "it's much better than CSLA", somewhere after the third feature I finally understood that the m in the name stands for "much better than CSLA".

I guess that there is always room for more codegens but I am not sure about general applicability of this one.

### Session 5 - Continuous Integration and automatic testing with Team Foundation Server (bu...bu...bui...bui...build...build...)

I haven't had the chance to use TFS so far, I use [Hudson CI](http://hudson-ci.org/) and [TeamCity](http://www.jetbrains.com/teamcity/) and wanted to learn more about TFS but I was worried that this presentation would leave me with a foul taste in my mouth because of bad presentation and lack of experience with other CI tools, boy was I right.

I must say that in all this time, all the articles/papers/books that I have read, and all the videos/live streams I have watched, I NEVER heard anyone use word "build" so much in such a short period of time. I understand that the topic requires it, that the speaker was nervous, but this was overkill.

The whole session was almost cut short because the speaker forgot to bring the power brick for his notebook, 5 minutes into the speech, he declared that he has only 15 minutes of battery left.
Somehow they managed to conjure one from thin air and the session continued.

He proceeded with explanations of CI and why it should be used, and how it works in TFS. Unfortunately to me it came across as "CI should be used to relieve the developer of stress of running test locally, because there is no way to group them and execute only one group, that is why you define a build for each set of tests you want, and let the TFS loose on their ass", somehow this does not sit right with me.

Next doosey was mentioning of 10 gigabyte code base, at which moment there was a lot of people thinking WTH are you guys building, then again, I am just a web developer, what do I know?

After some more "build" usage overload he proceeded to practical demo, then two things happened:

1. he connected via RDP to another machine running VMware instance in full screen, which by itself would not be that big of a deal, but the VM was paused, and the RDP bar was overlapping the VMware bar on the top of the screen. After a minute or two of trying to hide the RDP bar so he can press Play icon on the VMWare bar, he managed to do it with help of two other people

2. he then noted that he was "playing" with the VM last night and installed a developer only version of Visual Studio which does not have testing features, so he will not be able to demonstrate running test on the build agent machine, but we can imagine what he described of the process, and that he will demonstrate the building only.

From then on it went downwards, one moment he is explaining that the branch-per-feature is evil, and then he is explaining the branching setup for large projects like sql server which sounds a lot like branch-per-feature.

Personally my biggest shock was realization that on build machine I need to have Visual Studio installed. (At least that is how I understood it)

As the session came to an end, and we moved into Q&A; time, it became painfully obvious that it would be pointless to ask the speaker about comparison to TeamCity, and what would be a compelling case to move to TFS, aside from VS integration, and that it's an MS product.

In summary this was one of the scariest display of the NIH/MS Way/Before MS did it, it was not done/The only way right way is MS way up to date that I had witnessed

### Summary and thoughts

This was my first attendance on an event like this, I must admit I had high hopes, luckily after this, they will be lowered appropriately.
The effort to organize CodeCamp is great, and I really appreciate it, but is it possible that most of the sessions were ad-hoc, literally put together few hours before the event's beginning, all along with bad code, unprepared speeches, and fumbled setups?
I understand that the attendees are not what you would call "A list" audience, but we could use some professionalism on the stage and in the review process.

On the plus side I met [@nemke](http://twitter.com/nemke) and his coworkers, they are really a great bunch, I had a blast hanging out with them.
