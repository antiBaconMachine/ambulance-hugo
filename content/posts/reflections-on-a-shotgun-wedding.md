+++
author = "Ollie Edwards"
categories = ["dev", "iOS"]
date = 2014-10-15T10:07:55Z
description = ""
draft = false
slug = "reflections-on-a-shotgun-wedding"
tags = ["dev", "iOS"]
title = "Reflections on a shotgun wedding"

+++

I am a "web guy". I tend to refer to myself as a "Web developer", not a programmer, or a software engineer. I like making web apps.

Over the last year this has extended to making cordova apps for touch devices. This has entailed dabbling in some native code for plugin wiring, but essentially I'm still a web guy at heart. Javascript is my bread and butter, and I'm just fine with that.

It was therefore a bit of a shock then when recently I was tasked with making a fully native iOS app. My first thought was along the lines of.

>Well Shit.

The list of issues I have to begin with is reasonably daunting:

1. Whilst I'm at least proficient in a decent selection of languages, not a one of them is C or any C derivative.
2. I've never owned an iPhone, I'm not at all familiar with what an iOS app is supposed to feel like.
3. I implicitly mistrust any platform which forces you to use an IDE, let alone a specific IDE.

So how did I fare?

###The Language(s)

When this project dropped into my lap, Swift was still in the pipeline so I rolled up my sleeves and pulled out the big book of Objective C.

The syntax may be all weird but with ARC we basically have all the essential features of a modern language. I still felt like I was writing more for nostalgia then functionality, like a hipster composing a novel on my Grandfather's typewriter, whilst everone else just used their laptop.

Obviously Apple are attempting to address this with Swift. Xcode 6 dropped midway through this project so I was able to write a few isolated unit tests in Swift and the development experience was a lot more comfortable for me. 

In any case, turns out I've now been doing this long enough not to be bamboozled by syntax and I appear to have entered the phase of my career where I say things like

>Meh, a language is a language.

and keep a straight face.

So far, so good.

###The Interface

First I was excited. Drag and drop interface building that actually works? Then I was disappointed because it totally doesn't work.

If you want to build a single page app backed by a single controller then sure it's quite liberating to just chuck it all up on the page. Now make a second page using the same template layout. You can do this, but only in code, and the resulting layout will not render in interface builder, thus making interface builder useless. You're back to tweak and run. Xcode 6 claims to have fixed this with IBDesignables but as yet I haven't got this to work.

If you want to build a flexible reusable layout then you're doing it in code, this much I don't think is up for debate. Whether or not this is a good thing is. As a web guy I feel like separation of style and content has been beaten into me. Putting style code in the controller makes me cringe. But it is type checked. There are some clever tricks to make Objective C behave more like a style language, particularly with the constraints language.

When it comes to responsive layouts I don't think there's much to choose in terms of obtuseness between CSS media queries and autolayout. Both are functional yet maddeningly frustrating when they want to be.

Using autolayout reminded me of being a junior and just adding `position: relative;` to everything and crossing my fingers. I frequently found myself hitting `update constraints` without really knowing what it was doing.

As far as iOS style conventions, the interface builder does a great job of assisting you to lay out elements with nice consistent margins. Unfortunately as the language of IB is 'non human readable' xml, rather than the code we humans write, it's hard to play and learn by reading the output.

The most striking thing about all this is that I found myself saying

>Man, I miss CSS.

which honestly I never thought I'd say.

###Xcode and the Apple way

Having a closed platform means a consistent bug free experience for everyone. Except when it doesn't. Having a closed platform means all development effort can go into making one great IDE for all. Except when the IDE doesn't contain basic features like inline git diffs or organise imports and the keymap is all backward.

Other than interface builder, which I've already decided I can't really use in a sane way, what is Xcode bringing to the table? The only thing I can think of is code signing identity management, and the only reason that's even an issue for me is because Apple have made the process so complicated and opaque that I need an IDE to fix it for me.

Once you accept all its flaws I suppose it's nice that Xcode "just works". Except when it doesn't.

I do also appreciate the aggressive deprecation Apple can achieve in iOS, but this does have the disadvantage that so much of what you end up reading on the Internet is woefully out of date.

As my colleagues will attest my most common utterance in this project was probably

>Oh FFS, Xcode has crashed again.

###Conclusion

I spent the first few weeks as an iOS developer in a frustrated rage and I went home every day miserable. Then I got the hang of it. Much like any other technical discipline really. 

If I were to invest as much time in the platform as I have in web dev I'm sure I could make some rad apps and earn more money.

As the cliche goes there's more that unites us than divides us, which somehow was still a surprise to me. 

So, given the opportunity, would I make another iOS app?

>Fuck no.

Beyond the nagging differences of opinion, I simply don't think it's a good idea to be making the kinds of things I make, i.e. network bound web apps, that only exist on one platform. We already have an imperfect but usable system for creating cross platform content, it's called the world wide web, you may have heard of it. If you need to interface with hardware which HTML 5 does not yet support we have cordova.

Most importantly, I didn't find it fun.

The best thing that's come out of this experience is that I know I'm in the right field for me, and when I finally get this app out the door I'll run away as fast as I can with a smile on my face, knowing I'm exactly where I'm supposed to be.