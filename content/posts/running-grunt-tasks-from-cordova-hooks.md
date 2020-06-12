+++
author = "Ollie Edwards"
date = 2015-05-12T10:12:40Z
description = ""
draft = false
slug = "running-grunt-tasks-from-cordova-hooks"
title = "Running Grunt tasks from Cordova hooks"

+++

Cordova hooks are an invaluable tool for customising your build process. The canonical way to write them according to the Cordova documentation is to write them as node scripts for optimum cross platform compatibility. This is sound advice but for more complex projects we sometimes require a more structured tool. The obvious choices in the node space are Grunt and Gulp. In this post I'll explain how I recently adapted an existing Grunt build to be triggered using Cordova hooks.

####The basics

This solution uses a single `hook.js` script which should be placed in the top level `hooks/` and a `Gruntfile.js` residing in the top level of a Cordova project.

The general strategy on executing the hook script is

1. Get the name of the currently executing hook from the hook context
2. Require `grunt`
3. Execute a grunt task with the same name as the hook
4. Iterate the current platforms calling a grunt task of format <platform>.<hook_name>

Which looks something like this

```language-javascript
#!/usr/bin/env node

module.exports = function (context) {

    var grunt = require("grunt"),
        Q = context.requireCordovaModule('q'),
        minimist = require("minimist");

    var deferral = new Q.defer();

    var tasks = [context.hook];
    context.opts.cordova.platforms.forEach(function (platform) {
        tasks.push(platform + "." + context.hook);
    });

    grunt.tasks(tasks, minimist(context.cmdLine.split(" ")), function () {
        deferral.resolve();
    });

    return deferral.promise;
    
};
```

Couple of things to note here:

* We use the Cordova provided Q promise library so that the Cordova build will wait for the hook to complete before continuing.
* We read the command line args using minimist and pass these through to grunt as options so that we can parameterise our build.

#### Wiring up hooks

To get this script to fire you'll need to add a line to your config.xml for each hook you want.

```language-xml
<hook type="after_platform_add" src="hooks/hook.js" />
<hook type="before_prepare" src="hooks/hook.js" />
<hook type="after_prepare" src="hooks/hook.js" />
<hook type="before_compile" src="hooks/hook.js" />
<hook type="after_compile" src="hooks/hook.js" />
...
```

#### Ignoring missing tasks

What we have so far is grand as long as each and every task is implemented in your Gruntfile. This means that given the hook `after_prepare` running on platforms `ios` and `android` we'd need grunt taks `after_prepare`,`ios.after_prepare` and `android.after_prepare`.

This is suboptimal, it would be much better if each task was optional. You can achieve this by running the tasks one at a time and wrapping them in try catch blocks. I've achieved this using my own [grunt-graceful](https://www.npmjs.com/package/grunt-graceful) plugin which does exactly this at grunt level thus keeping our hook script a little cleaner.

```language-javascript
var prefix = "graceful:";
var tasks = [prefix + context.hook];
    context.opts.cordova.platforms.forEach(function (platform) {
        tasks.push(prefix + platform + "." + context.hook);
    });
```

#### Conclusion

In this post

* We've seen how to programmatically execute grunt tasks
* We've triggered grunt tasks from a Cordova build
* We've passed command line args between the two tools

For complex projects I think grunt can help enormously to organise our build and accelerate development. In javascript land there are very few more complex projects then a multi platform Cordova web app. I think that makes these two technologies a natural fit and with only minimal glue code we've made this happen.
    