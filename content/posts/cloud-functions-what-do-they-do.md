+++
title = "Serverless, what is it good for?"
subtitle = "Sand is overrated, it's just tiny little rocks"
date = 2020-09-16T15:12:24Z
tags = ["cloud", "serverless"]
draft = false
+++

A colleague asked me the other day "what is serverless actually good for" and it did take me a minute to come up with a good answer. There seemed to be a lot of hype over lambdas and cloud functions a few years ago, it felt like we were at a tipping point in the cloud journey where server boilerplates were going to be abastracted away from us just as the phhysical servers were abstracted before. No one would have asked this question because serverless tech was obviously going to be good at everything. 

It hasn't really panned out that way in my opinon but I have still had several great experiences. I have by far the most experience with google cloud functions so I'll refer to them for the rest of this post but lambda is very similar and there are plenty of other providers with equivilent offerings.

The good:

* Cheap
* Low boilerplate

The bad:

* Slow
* Limited resources

The best thing about serverless is undoubtedly the price. GCF pricing is reasonably complex to figure out because it it based on invocations, memory _and_ cpu usage which are all calculated separately, The good news is the free tier is very generous, which means for low volume services you can effectively run for free.

Cloud functions are unsurprisingly based around the function as the primitive unit so instead of writing a whole server boilerplate you just write a handler function. Now the choice of languages that GCF support aren't exactly boilerplate heavy languages anyway but it's still fun being able to throw these things up quickly.

The biggest downside for me is request latency, particularly if the function is 'cold' it can take seconds to complete which is not really acceptable for a user facing api for instance. If it's for a CI/CD system, you probably won't notice.

Resources are clearly quite limited as well. For a nodejs GCF you have 256Mb of combined ram and disk space to work with, and bare in mind the runtime takes ~80Mb of that immediately. One of the first projects I did with GCF was part of a an artifact publishing pipleine. This entailed pulling down a tarball, unpacking it, processing the content, repackaing it and shipping it on. Certain publishes also came in multiple flavours which meant the available disk space was very quickly used up. With careful sequencing and using streaming transforms where possible I managed to work around the issue but do bare in mind that if you are dealing with large files serverless may not be the right choice for you.

The most successfull projects that I've worked with cloud functions on have all been CI/CD pipelines. This works because this style of pipleine is

* Typically low volume compared with user facing services
* Not especially latency sensistive

Even a pipeline triggered 5 times a working day by 100 developers is still only 10k invocations a month. The free tier of GCF incluides 2 million invocations a month. They might take a couple extra seconds to run but typically we don't sit staring a pipeline waiting for it to finish (right?) so I'm not too fussed. If you have a complex pipleine to build and out of the box CI/CD tooling isn't getting it done it's well worth giving it a go. There is however an argument that if you have headroom on existing compute resources you are already paying for then you should use that headroom first.

Like most promised revolutions in tech serverless has never quite lived up to the hype for me but it still makes for a good tool to have in your pocket for certain use cases. What use cases have you found?
