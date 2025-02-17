---
layout: post
title: "A retrospective rant on software design in particle physics"
comments: false
description: "I have a lot of opinions."
keywords: "software, design, particle physics, open source"
---

This rant was originally started in a draft of a post for my DeepTransfer project.
It quickly ballooned but I realized I still had a lot more to say so I've created a dedicated
post instead. I also reformatted the presentation as several case studies of specific instances
where underlying dogmas have created software that is hard to use, necessitating that the user
become an expert in a topic that is entirely uninteresting. Namely, one often has to become an expert on
how to finagle what they want out of software that is obsolete outside *this specific field of study*
(or even just this specific collaboration!) because the world outside has better options that are more
intuitive.

I admit that my words can be a bit harsh but I decided not to dampening
the tone because it's the manifestation of a lot of my frustration. To tone it down
would be to muddle the message that I *want* the field to improve. And I think it would if it
treated software as a fundamental experimental tool on equal footing with all of the components
of the detectors. This is also not to say that there aren't people doing good work
and writing good software (the younger generation of ROOT devs are doing cool work to modernize things) - but
I've sat through enough meetings meant to discuss new software ideas that I can tell you very few lessons
are being learned from previous (unacknowledged) mistakes.

I also admit that this may be a case
of just staring long enough to start seeing cracks - I'll probably develop similar feelings
towards non-particle physics software once I have to use it for an equal period of time.

Finally, I've found this process of writing these case studies helped me form a useful development philosophy that
if you want to convince people to use your product (software or otherwise), you must make as few
assumptions about your user as possible. For them, using your product should
feel obvious and effortless.
No one cares what clever engineering you did to make your product 10%, 20%, 30% better
if the overhead is that you made assumptions of the user, which for them, has made your product hard to use.

-------------------------
## CERN's ROOT
-------------------------
- More recent developments are good but built on a backend trying to satisfy python via C++ mappings(? wrapper?)
- RDataFrame a great example - fast because it runs C++ code. If you want to use it in python, you write python strings of C++ code.
--- IDE can't code highlight your work
--- Python users with little C++ experience make common mistakes like requesting elements of vectors that are outside of the vector (leads to seg fault which gives no useful info)
--- Anything more than a line of data manipulation in C++ is beyond the user ability of someone working in python
- Other small things like histograms being indexed at 1 (so that the 0 index stores underflow)
- ROOT's memory management is in direct conflict of python's memory managements
--- Object being preserved by ROOT will be tossed by python and when ROOT finally wants to use it, you'll get an unintelligible seg fault (RooFit is the biggest culprit of this - pull ROOT forum post I made on it)

-------------------------
## CMSSW
-------------------------
- cmds done run in the order they were defined (declarative nightmare)
- so bad it got wrapped by cmsdriver.py
- which is also so bad that cmsdriver got wrapped by runTheMatrix
- global tags have no easy to access versioning
- CMSSW environments have no easy to access versioning
- cmsRun ingests config but error messages won't tell you what you broke in the config if there's a mistake (seg faults with no useful output)
- incompatible with python 3 + ROOT (compiled for python 3)


-------------------------
## NanoAOD-tools
-------------------------
- stupid line length formatting
- incorrect code sitting in repo for years (being ignored when brought to "experts")

-------------------------
## Faster analysis tools
-------------------------

Several years ago, I was considering different faster analysis tools for CMS data analysis.
There were (and still *are* really) two competing options:

- RDataFrame which takes its namesake from
the pandas DataFrame albeit with all of the positive and negative attributes of ROOT dragged with it
(positive being that it's designed for particle physics workflows and negative being that the designers
were particle physicists who want to make everything from "first principles").

- Uproot + coffea which "uproots" itself from the dependencies on ROOT so that it can use
open source tools written by software engineers (yay - no remaking the wheel!) but often wrapped in software that is still
designed by particle physicists (...no not like that!).

I went with RDataFrame for the simple reason that it was ROOT native and I wanted others to use the tool
I was working on ([TIMBER](https://lcorcodilos.github.io/TIMBER/)) and the number one way to 
get your code into other people's code is to make the transition both advantageous *and* seamless.

The *seamless* part is where I had a hard time with uproot and coffea. The "Getting Started" sections
of the respective packages are... hard to follow. And that was an opinion I heard echoed from others - specifically
by people in leadership positions that could encourage/discourage the adoption of such tools.
I don't think I spent more than 5 minutes on each "Getting Started" page before I realized RDataFrame was the obvious choice - 
"better the devil you know" in some aspect.

From then on out, I projected I would never touch either uproot or coffea - I had no need and realized I
probably wouldn't be around the field long enough for a need to develop. That is until I decided on this DeepTransfer project...

--------
# Other thoughts
--------
I've taken two very different positions on ROOT - high energy physics' (HEP) dominant 
computing software - before and after finishing grad school. In the final several
years of my program, I was an advocate for keeping computing in the ROOT
ecosystem. In contrast, after graduating, I couldn't wait to start exploring everything *not* ROOT
available with python. Plotly, seaborn, numpy, pandas, Dask, SQL, Parquet - all made
me say, "Wow, what have I been missing?" I even started drafting a post about
all of the nonsensical design decisions in HEP software, ROOT included.

I'll release that eventually but a recent foray into decoupling data from ROOT
formats and tools has me learning that, despite it's... *idiosyncrasies*...
ROOT sits on the throne of HEP software for a reason.

---------

## Opinions formed from TIMBER

I wrote and released (and still support) a python library called [TIMBER](https://lucascorcodilos.com/TIMBER/)
a couple of years ago. It's essentially a wrapper for ROOT's RDataFrame that uses
a directional data processing graph to organize data manipulations in a traversable structure.
The meta information stored in the tree makes it possible to have high level
manipulations that consider the context of the given node being manipulated.
It makes analysis code more succinct and easier to debug, encourages better general data practices, standardizes
common algorithms, and runs *fast* because of the RDataFrame back-end which is lazy, only executing
at the last moment (filling a histogram, writing out data, etc).

When I was initially developing TIMBER, I considered a different back-end based on `uproot`,
an I/O python library that is meant to "uproot" the user from ROOT dependencies.
A whole community of physicist software developers (largely based around IRIS-HEP but in other places as well)
have started basing tools on `uproot`, making a lot of effort to attract people away from ROOT.

Most physicists are aware that HEP *used*
to have the biggest data sets in the world but are now beaten by orders of magnitude by the likes of Google, Amazon, and others
who have put a lot of money into building out tools for efficiency in processing data. So it's only obvious
to start looking around at what everyone else is doing!

But building a tool *without* ROOT has a very obvious downside that is largely ignored by these physicists-turned-software devs:
**HEP technical knowledge is concentrated almost entirely in ROOT**.

Building a new tool is easy. Convincing people to use your new tool is really hard.

And that gets ignored **a lot** in HEP. My whole design philosophy with TIMBER was to
make it as easy and obvious as possible for physicists to start using it - and that meant
tying it to ROOT which 

1. wouldn't disrupt the pipelines that come before and after TIMBER, and
2. make the back-end an already familiar ecosystem.

I even used the default Doxygen theme so TIMBER docs would *look* like a familiar place
when a user inevitably needed help.
