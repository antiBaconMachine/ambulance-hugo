+++
author = "Ollie Edwards"
categories = ["dev", "javascript", "grunt"]
date = 2014-05-22T12:14:04Z
description = ""
draft = false
slug = "adding-a-task-graph-to-grunt"
tags = ["dev", "javascript", "grunt"]
title = "Adding a directed task graph to grunt"

+++

Last year we migrated our build system for a multi platform cordova app to Grunt from ant. It was great. Well, actually it was just quite good. See, whilst I'm really not a fan of ant, the ant task graph is nice. It upsets me that I can't express branching task dependencies in Grunt.

####The problem

In this case we wanted to do a bunch of common setup once then do n platform specific steps for n platforms. The idiom we settled on is something like this:

```language-javascript
grunt.registerTask('common', [
 	'taskA',
    'taskB'
]);
    
grunt.registerTask('eachPlatform', function() {
 	['foo','bar'].forEach(function(thing) {
       	grunt.task.run([
        	'setCurrent:' + thing,
            'taskWhichDoesSomethingWithCurrent'
       ]);
    });
});
    
grunt.registerTask('setCurrent', function(current) {
    grunt.config('current', current);
});
```
    
Pretty verbose no? Also WTF is all that setCurrent about? Well it's a hacky way to get a task which changes the value of a grunt config param into the task queue.

This works well enough but is hardly intuitive. If each platform also needs specific setup/teardown (which they will), then that's an extra problem to solve. You'll end up just throwing the whole thing out the window and building n different task stacks for n platforms and just accept you're doomed to be soaking wet.

On the other hand in evil ant land, we're reasonably dry and depressingly readable
```language-markup
<target name="ios" depends="taskA, taskB"/>
<target name="android" depends="taskA, taskB"/>
```    
####So

I got to wondering how hard it would be to add an acyclic directed task graph plugin to grunt which would allow us to express these kinds of dependencies in a vaguely ant like way but still take advantage of all the things we love about grunt.

Being node land there are of course already plenty of directed graph libs in the npm registry to pick from so the actual behind the scenes implementation is fairly trivial. I used [graphlib](https://www.npmjs.org/package/graphlib) which has both the data structure and algorithms necessary to build a task graph. I'll save the detail on how I implemented the guts of the plugin for another day as what I really want to talk about here is the interface.

I didn't want the graph to be an entirely new build system, I wanted it to play nice with existing grunt infrastructure. I also wanted the configuration of a graph to be as natural as possible to grunt users. The obvious solution seemed to be to implement this as a multitask:

```language-javascript
graph: {
    c : {
        dependencies: ['graph:b',],
        task: function() {
            console.log('C');
        }
    },
    b: {
       dependencies: [ 'graph:a'],
       task: function() {
           console.log('B');
       }
    },
    a: {
        task: ['graph:a_inner']
    },
    a_inner: {
        task: function() {
            console.log('A');
        }
    }
}
```
    
Which I then coupled with a `run-graph` task so I can run the graph tasks like so

    $ grunt run-graph:c
    Running "run-graph:c" (run-graph) task

    Running "graph:a" (graph) task

    Running "graph:a_inner" (graph) task
    A

    Running "graph:b" (graph) task
    B

    Running "graph:c" (graph) task
    C
    
We can pass multiple tasks in any order and the graph will take care of dependencies so `grunt run-graph:c:b:a` will still execute in order `a,b,c`.
    
I'm fairly happy with this as a proof of concept. Check it out for yourself at [grunt-digraph](https://github.com/antiBaconMachine/grunt-digraph.git) or dive right in with `npm install grunt-digraph --save-dev`.

####Improvements

There's a few niggles I've already noticed, the headlines are: 

* Need to look at a way to test this effectively, probably involves some sort of mock grunt object. How are others testing their non file based grunt plugins?
* Declare dependencies from the same namespace in a more concise way, something like `['foo', {graph: [a,b]}, 'bar']`. I don't think I want to expand this to a full object, as we'd loose the simple instantly readable list of tasks.
* A clean way to pass arguments to graph tasks

####Conclusion

This was a fun and pleasingly fast project. It's one of my first grunt plugins, and certainly the most complicated I've attempted, yet I found the process largely straightforward. It's incredible how the node community reduced this problem to essentially a wiring exercise between existing components. The [graphlib](https://www.npmjs.org/package/graphlib) library was excellent and did everything I needed to get this done fast.

The major issue I have with the grunt plugin scaffold is it doesn't have an obvious testing solution for tasks which deal with anything other than file IO. This is my most pressing issue to investigate going forward.

Whilst grunt as a build tool may be slightly limited for complex projects, particularly when compared to ant, the incredible community support in terms of grunt plugins and more general node libraries, more than makes up for these shortcomings. 

Do check out the source on my [Github](https://github.com/antiBaconMachine/grunt-digraph.git) and let me know what you think.