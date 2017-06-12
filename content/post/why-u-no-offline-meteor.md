+++
author = "Ollie Edwards"
date = 2014-11-16T17:30:04Z
description = ""
draft = false
slug = "why-u-no-offline-meteor"
title = "Why u no offline meteor?"

+++

Meteor 1.0 dropped a couple weeks ago which prompted me to revisit it. I'd previously dabbled with creating a race picker for the board game Twilight Imperium so we could spend less time faffing when it came to game day. Whilst I found it to to be an enjoyable experience, like most people I had concerns around code re-usability. I also felt for that particular app I didn’t really need the coolest feature of meteor, realtime by default.

With the newly minted 1.0 release I wanted to do something that actually leveraged the strengths of the platform. I settled on making a kanban board app. Yes there are plenty of existing solutions for this problem, but where is the fun in that? A kanban board actually has a fairly hard requirement on being realtime if it is to be of any use. I figured with meteor a minimum prototype should be pretty straight forward. And it was. Whatever my other concerns about the platform I haven't seen a faster way to get a realtime collaborative app up and running.

At this point I noticed another feature which had dropped in meteor when I wasn't looking, namely Cordova support. Now this started to excite me, wouldn't it be cool to have an offline first app so I could create cards on the go as they come to me and have them sync back to base when ready? Of course with Meteor's "database everywhere" paradigm, an offline app should be a piece of cake right? I mean the database is literally doing a 2 way sync anyway, surely I can just tell minimongo to cache the data locally in indexedDB and boom, offline first app with no effort, right?

Well apparently no. Minimongo doesn't seem to have this functionality, and surprisingly there doesn't seem to be much of a clamour for it. This does seem odd to me that the Cordova feature would have shipped without much indication as to how the app is going to work without at least an initial seed from the remote database.

There are 2 projects I'm aware of, attempting to deal with this problem. One of them has been abandoned, the other [GroundMeteor](https://github.com/GroundMeteor/db) is more promising. It operates at collection level and uses localstorage as a backend. 

GroundMeteor certainly looks like an interesting and useful solution but it rankles that I can't sync a whole dataset as I can with [PouchDB](http://pouchdb.com/). There's some interesting frameworks emerging in the offline space, with [Hoodie](http://hood.ie/) at the vanguard. Whilst Meteor does appear to inhabit the other end of the spectrum with it's focus on realtime, the infrastructure necessary to do both well does seem to be in place. As soon as I get the kanban project off my dock I'm going to dig into this.

Congrats to the Meteor team on reaching 1.0