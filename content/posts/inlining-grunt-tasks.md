+++
author = "Ollie Edwards"
categories = ["dev", "java", "grunt"]
date = 2014-05-27T15:57:51Z
description = ""
draft = false
slug = "inlining-grunt-tasks"
tags = ["dev", "java", "grunt"]
title = "Inlining grunt tasks"

+++

Grunt config files can easily become very verbose and disparate with related tasks scattered far apart. Without careful organisation and maintenance they can become quite unwieldy. In this post I'll discuss some of the ways I've been experimenting with to make gruntfiles terser and keep related components together. I'll introduce the [grunt-inline-task-sugar](https://www.npmjs.org/package/grunt-inline-task-sugar) plugin which I created to implement some of the ideas discussed.

#### Anatomy of the alias task

The grunt 'alias' task type is probably familiar to you:

```language-javascript
grunt.registerTask('release', [
	'build',
	'minify',
	'copy:someFileToSomewhere',
	'shell:someShellCommand',
	'gargle',
	'rinse'
]);
```
    
Arguably this is as much information as you should ever need. The task list is nice and readable and because it's just a dumb array you can't really make it any more complicated then this.

On the other hand, which files is that copy task operating on again. What shell command is being run? Worse, when you look at the generated artifacts you find unmagnified code. What's going on? turns out that minify task was declared with a conditional

```language-javascript
grunt.registerTask('minify', function () {
	if (grunt.option('minify')) {
        grunt.task.run([
            'useminPrepare',
            'concat',
            'uglify',
            'cssmin',
            'usemin',
            'clean:minified'
        ]);
    }
});
```

So to get the minify task to actually run you need to pass the option `--minify`. 

To find out all of these things you need to scroll to several different places in the Gruntfile. Of course these kinds of problems can be alleviated with solid, well maintained documentation, but we all know that is highly unlikely to be available.

#### Inline conditionals

A common idiom I've developed is to use the array functions from [lodash](http://lodash.com/) to preprocess the task list.

```language-javascript
grunt.registerTask('default', _.compact([
    'jshint',
    (grunt.option('noTest') ? null : 'test')
]));
```

The `_.compact` function removes all falsy values from an array. I'm using the ternary operator to do the conditional check and return a falsy value when it fails. This results in a task list which does not contains `'test'` when the `--noTest` option is passed. 

#### Nested arrays

Similarly, it can be useful to conditionally execute whole nested arrays.

```language-javascript
grunt.registerTask('default', _([
    'jshint',
    (function() {
    	if (someCondtion) {
         	return ['foo', 'bar'];
        } else {
           	return ['spam', 'eggs'];
        }
    }());
]).flatten().compact().value());
```
In this case we first need to flatten the nested arrays which we do with `_.flatten` function. As we're using two lodash functions back to back we use the chain syntax which means we need to call `.value()` at the end to get the unwrapped value (see [chain doc](http://lodash.com/docs#_)).

#### Making it transparent

The [grunt-inline-task-sugar](https://www.npmjs.org/package/grunt-inline-task-sugar) plugin abstracts these features into the `grunt.registerTask()` function. Once it's loaded you can pass 'smart arrays' unwrapped to the registerTask function.

#### Inlining multitasks

Whilst writing this plugin I threw in an extra experimental feature to inline  multitask definitions. Arguably, defining simple multitask blocks in the grunt config init then using them exactly once in an alias task in a disparate location is a bit messy. I've therefore added a feature where any objects encountered when parsing the task list will extend the grunt config via the excellent [grunt-extend-config](https://www.npmjs.org/package/grunt-extend-config).

```language-javascript
grunt.registerTask('helloWorld', [
    'someTask',
    {
        shell: {
            hello: {
                command: 'echo HELLO'
            },
            world: {
                command: 'echo WORLD'
            }
        }
    },
    'someOtherTask'
);
```

This will register the tasks `shell:hello` and  `shell:world` and register the task  helloWorld as an alias to `['someTask', 'shell:hello', 'shell:world', 'someOtherTask']`.

However, I'm not convinced this is a sensible alternative to good naming conventions:

```language-javascript
grunt.registerTask('helloWorld', [
    'someTask',
    'shell:echoHello',
    'shell:echoWorld',
    'someOtherTask'
);
```

We can't tell for sure what the two shell tasks are doing, but we can make a fairly good bet based on their names. Maybe this is enough for your build, and you needn't bother with inlining. 

#### Conclusion

I've discussed some ways of making task definitions terser and more contextual. Arguably this has been to the detriment of readability. A big factor in the success of Grunt has been its relative simplicity, but this does lead to maintenance issues for large projects. It's up to you to decide the right path for your build, feel free to flame my approach entirely in the comments below.

