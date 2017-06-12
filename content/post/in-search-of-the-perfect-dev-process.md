+++
author = "Ollie Edwards"
date = 2015-06-26T14:58:16Z
description = ""
draft = false
slug = "in-search-of-the-perfect-dev-process"
title = "In search of the perfect dev process"

+++

Professional development is a very different game to the late night hack sessions where we first learn the ropes. The first couple of years of my career were really more about learning how to communicate about code than learning to code. I've been involved in a lot of different processes, some very ad hoc, some with buzzword names and a rulebook. All of them have taught me something, but above all what I now know is that there is no perfect process.

Scrum and Kanban seem to be the two major competing methodologies these days and both have interesting qualities. I believe the most important thing these two systems have going for them is that they are flexible and encourage incremental process improvement within a team. With that in mind I'd like to describe some of the key features I like to see in a process. 

###A board...

Scrum and Kanban both rely on the use of a board with columns or rows representing the flow of work through the process. I have always liked boards, I've become a better more organised developer since learning to work with them. 

A physical paper based board is good for prototyping but a digital board is far superior as it allows for the team to work from anywhere. Modular tickets with many subtasks are much easier to achieve with a digital system. A single combined system which also serves as the general purpose issue tracking system is also invaluable.

###...which stays accurate

Kanban is built around the notion of flow whereby everything moves forward inextricably toward the end product. This is understandable given it's genesis in physical manufacturing. A component made on a production line either passes acceptance and is shipped or fails acceptance and is discarded. It does not fail acceptance, move back to the start and get remade.

Software is not physical. A ticket which fails QA is a ticket which requires more development. Common Kanban ways of representing this are by putting a red flag on the ticket then carrying on development... Whilst the ticket is in the QA column... even though there is a column clearly marked "development" right next to it. No. That is counterintuitive, unhelpful and downright stupid.

You can tell me the ticket is still in QA until you are blue in the face but we all know the ticket is actually in development. Your board is now lying. Wilfully. If your board is lying then you are lying. To yourself. Why are you lying to yourself? What are you achieving by making the board more opaque?

There are two nice easy solutions to this:

1. Just move the ticket to it's logical column
2. Create a subtask to fix the issue and run this ticket through the board

I'm not fussed between these, as long as the board sustains an accurate sane reflection of reality I'm happy.

###A daily meeting

Sometimes I leave a standup wondering why I bothered going. The one time in ten I actually gleam some important information makes the other nine worthwhile. A boring standup just means you already know what's going on in the team and things are running smoothly. Good for you.

###Variable length iterations

Scrum divides work into logical units of time, the sprint. This is a great tool for generating momentum in a dev process. Having a clear goal to aim for at regular intervals generates a real rhythm in a team and helps avoid end of project crunch time.

However, given a fixed size container, sprint planning can be tricky to balance. A single feature is rarely exactly sprint sized and we end up making awkward divisions in work just to suit the container size.

Instead I far prefer the approach of hitting one feature or small group of related features for as long as it takes to get them done. A common criticism of this is the lack of predictability this offers. I'd argue that having an effective estimation strategy will mitigate this and over time unusually sized iterations will average out.

###Clear (non) ownership of work

Ownership means different things depending on how the system has been implemented. Is the owner the technical lead responsible for the whole ticket or is it the person currently working on the ticket?

For me the board should be most useful to people actually doing the work. When I look at the board I want to get a clear visualisation of what I need to do now and next. Ownership should therefore be about developers not project managers.

Kanban sidesteps the issue altogether by doing away with ownership. There's no reason why this kind of flexibility can't be used alongside a more formal ownership paradigm.

My preferred ownership scheme would follow these rules:

1. Work MAY be pre assigned to developers at planning time. This allows them to mentally plan their workload over this iteration.
2. Unassigned work can be picked up by anyone. Typically this is low priority bug fixes, not the meat of the iteration.
3. If more than one person is working on a ticket then each person should have their own sub ticket. There's no restriction on creating and assigning additional subtasks.

I think mixing elements of both systems allows for developers to start each iteration with a good idea of what they'll actually be doing, but also have flexibilty to pick up additional work as needed.

###Push not pull

In Scrum tickets are moved to the next column when a stage is completed. In Kanban the tickets stay where they are and an extra flag is added to them to indicate they may be pulled to the next column. For me this distinction is academic. As long as there are clear ownership rules in place the Kanban style flag is just an added complication which adds no value. When you finish a ticket move it to the next column and (un)assign it. Done.


###Soft or no WIP counts

A key feature of Kanban is the work in progress limits placed on each column whereby no more than n tickets may enter a column. WIPs are an interesting idea but if you're going to use them you need to be very clear on the objective behind this. The common justification behind them is that "help identify bottlenecks". I can help you identify bottlenecks without WIPs right now. Go and look at your board. You see that column with lots of tickets in it? That's your bottleneck.

In fact I'll do you one better. Your bottleneck is QA. How did I know? Because it's always QA. That's not an insult to the many great testers I've worked with, it's just that good QA is really hard and takes a long time.

A more generous reading would be that WIPs prevent bottlenecks getting worse by forcing people to shift their priorities to help relive the bottleneck. That is admirable but again a sprinkling of common sense is more valuable here than a dogmatic set of rules. If you see a column creaking under it's own weight, then do something about it, it's your process, it's only going to hurt you if you let it collapse. If you really need a big red number hovering over your head to get you to do the right thing, you may be in the wrong business.

WIP counts also play very badly with subtasks which as I've established I'm a big fan of. When the rule changes subtly from "only n tickets here" to "Only n tickets except those little blue ones which you can have loads of" the usefulness of the rule is eroded. 

If you don't use subtasks then in order to keep your board sane you'll need to move tickets backward. This also doesn't work well with WIPs as the rule is now "only n tickets here unless one moves backward in which case I suppose that's ok".

I think WIPs are a great academic theory. I am however still waiting for the first time they do anything at all useful for my process. I can however list numerous times when they've been actively harmful, obstructive and generally wasted precious development resources because people were literally unable to work due to blocked columns. When your process is actively preventing progress something has gone seriously wrong.

If you really must use WIPs, most of these issues can be mitigated by making WIPs advisory. Honestly though, what's the point?

###A QA column

This one almost goes without saying, any decent development process should have a formal QA stage.

Ownership of QA is something I struggle with. I have two approaches I like.

1. All QA tasks are sub tasks of the ticket. This means the original developer maintains ownership of the ticket whilst the ticket is in QA so that there is clear point of contact.
2. The ticket is moved to the QA column when complete and unassigned, ready for a QA to pick it up or assigned to a specific person. This keeps the board uncluttered but loses a little bit of the chain of responsibility. In practice this is only an issue for paper boards as any decent digital system will have an audit trail to show who worked on the ticket.

### Conclusion

Devs like process. Process make Devs happy. Except when process introduces confusing, opaque and obstructive practices.

If a process is not working for you there absolutely has to be a mechanism for you to feed this back to the rest of the team, generally in some sort of review. Clearly not everyone can have their perfect process at the same time but frank and honest discussion is still crucial. 

The process must primarily serve the people on the ground doing the work. Stick to that and it's hard to go wrong.


