---
layout: project
link: "ageiersbach/cmdr-log"
project: "CMDRLog"
tag: "cmdr"
---

[Elite Dangerous](https://www.elitedangerous.com/) is a game that has a pretty
extensive ecosystem of data, which makes
it an interesting subject for hacking. There are already a number of sites
which use and expose the data that the game provides. One of the cool features of
the game is that it keeps a json log of all your actions as a player, including
all the missions and items you pick up as you move through star systems.
CMDRLog is intended to mirror this log and provide some
useful integration with existing tools, like [EDDB.io](https://eddb.io) and
[coriolis.io](https://coriolis.io/). For example, in the game there
are mining (asteroids!!) missions which you can stack up, but the game does not
tell you if you've collected enough material to turn them in. Also, there are
frequently excess minerals and metals, which would be nicer to offload on a nearby
station for a tidy profit--within a reasonable distance, of course.
This requires tracking the cargo and missions, and pinging EDDB.io for nearby
stations to find out the highest sale. Preferably on a phone, with an interface
that doesn't burn your eyes out in a darkened room (the way EDDB.io does).
