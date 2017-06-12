+++
author = "Ollie Edwards"
date = 2014-12-30T12:46:44Z
description = ""
draft = false
slug = "roll-your-own-javascript-framework"
title = "Roll your own JavaScript framework"

+++

In this article I intent to talk, at high level, about some of the decisions we need to make when laying a out a green field JavaScript app. I'll address four key problems which most medium to large apps need to address:

* How the code is organised and loaded
* How isolated units of code communicate
* How the app presents
* How the presentation is structured

I'll discuss why I believe selecting small well designed components to address each issue individually is preferable to a large monolithic framework.

###Introduction 

Over the last few years we've got used to the next big thing in JS arriving with depressing regularity. In my opinion, and I don't think I'm alone, we've reached a saturation point where even if the framework to rule all frameworks dropped into our laps next week, we'd not even notice it amongst the deluge other tools oozing from hacker news daily. I no longer feel that angular, ember, flavourOfTheWeek.js etc are a sustainable option given that all will probably be obsolete in eighteen months time.

This year I've been increasingly adopting a roll your own approach utilising whatever micro frameworks I need on a project by project basis, and I want to talk a little about how I've been going about this.

I want to keep this post quite high level and not get carried away with dogmatically describing what your next app should look like. I don't want to provide template code on github because that's basically a framework but worse. I just want to outline my thought process when approaching an app from scratch. 

Essentially the four key concerns I outlined at the top boil down to:

* Modules
* Decoupled comms
* Templates
* Routing

There are tricky choices to be made in all of these categories. The main criteria I use to evaluate these components are:

* Easy component upgrading and substitution
* Ease of unit testing
* Comparable speed of development with an off the rack framework

### Modules

There are 3 common options here which are probably familiar to most people reading this post:

* commonJS
* AMD
* globals

The deciding factor for which solution you pick is likely to be the export format of other modules you want to use. No one enjoys shimming a library. For my money I'm pretty sold on using CommonJS modules with a web wrapper. The simple reason for this is that npm is hands down the most complete resource for JS libs available.

Web wrapper wise I've used [browserify](http://browserify.org/) and [webpack](http://webpack.github.io/), both to good effect. Webpack tends to shade it for me as it's module chunking system is invaluable for large web apps. It can also be used to load other non-js resources via loaders, though that's a feature that should be used with care to avoid cross contamination of style and content.

### Decoupled comms

Your modules are going to want to talk to each other. Any framework worth its salt is going to provide a mechanism to do this, so we now need to pick our implementation. 

The most common one is an observer pattern, typically implemented as a small mixin. This is a good start but my personal preference tends to be for a global mediator pattern.

Now, lots of people dislike mediators as they communicate with string messages instead of simply calling a function on a registered object. It's possible to create some serious spaghetti code by positing obtuse non descriptive messages around. You do however have ultimate flexibility in module composition as each module is completely isolated from every other module. 

In the observer pattern each module knows which other modules it needs to listen to. In a mediated system each module need only know the message it needs to listen for which is a subtle but important distinction. We can completely replace the module generating the message without changing a single thing about the listener. This is invaluable for unit testing and for cycling a project through test environments which may require different implementations of certain components.

I've found a great way to avoid spaghetti code, is to adopt clear conventions as to the kinds of messages that are sent, which are something like this:

* A module only ever sends a message to notify that it has done something. It may send a didProduceResult message, but never a didRequestResult message.
* A module should send messages which uniquely identify the sender as part of the message e.g. `controller:foo:didBar`. Typically the qualifiers should match the physical location on the file system to avoid clashes. This way we'll always know where a message came from.
* The mediator implementation should understand the message hierarchy. Using the previous example a module should be able to listen for a `didBar` message without caring if the actual sent message is `controller:foo:didBar` or `controller:spam:didBar`.

The last point is incredibly powerful if used properly and for me is probably one of the biggest differentiators when compared to the observer pattern as it's simply not something that can be achieved with observers.

For my money [mediator.js](http://thejacklawson.com/Mediator.js/) ticks all the boxes and is on npm. I should note though if you run straight off to try this out, the way the hierarchical messages have been implemented means you'll need to reverse the convention so the most specific part comes first e.g. `didBar:foo:controller` which actually makes more sense once you get your head around it.

### Templates

This one can get almost religious in nature and it's probably the best argument for rolling your own framework. There are a multitude of dialects, and a vast difference in feature sets. Unfortunately it's also the most important component to get right first time. If you switch template library mid way through a project then you're writing every template again from scratch. I tend to favour handlebars alikes which augment rather than replace HTML but others prefer the more terse options.

Another key distinction you'll want to consider is how much logic do you want in your template? I've seen and written some hideous mangled branching logic in a template which would really have been better in the model object. On the other hand sometimes just throwing a simple if/else in the template really is the most straight forward way to get something done.

Typically I also like working with 2 way bindings between templates and models. A good componenting feature is also extremely helpful in a templating library. This is really a personal preference area. For me I most recently used the Guardian's [Ractive](http://www.ractivejs.org/) library which is a reasonably heavyweight full featured library which certainly doesn't qualify as a micro library but It does provide everything I want and at great speed.

I will say that a well thought out library should more or less eliminate the need to do DOM manipulation. If you find instead of updating model objects you're updating a view, then something has gone wrong.

### Routing

Much more straight forward, almost every app needs a way of changing content pages. This is really a question of how many bells and whistles you need. [Director](http://www.javascriptoo.com/Director) is full featured but slightly larger than some other bare minimum solutions. There isn't actually too much more I can add here, it really is a question of doing you research and figuring out what you need. This is a prime example of something that should be easily replaced though, so start simple and upgrade as and when you need.

### Putting it together

The most important thing at this point is documentation. It doesn't have to be Magna Carta, but one of the biggest dangers when rolling your own is forgetting or never knowing in the first place what the design goals of the system where. Readme files often say what something does but not often why. Take extra care to explain the choices you've made and your future self and colleagues will thank you.

Probably the biggest time sink in rolling your own framework is getting a good working grunt/gulp file together to do all the minifying, shuffling and rewriting, which off the rack frameworks tend to do for us. It's true you're going to loose a bit of time here, but I'm convinced the long term maintainability of your app will improve as a result.

###Conclusion

I've tried hard not to provide hard and fast solutions here as the point of rolling your own framework is to have things your own way. I have made a few recommendations for the kinds of libraries I think work well in a DIY system. 

If you've never tried to implement your own framework, then I implore you to give it a go. I think you'll be surprised how far you can get with just a little glue code and a wonderful open source ecosystem. Even if you never use the result, I guarantee you'll be a better developer for the experience.  




 
