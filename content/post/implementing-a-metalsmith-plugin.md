+++
author = "Ollie Edwards"
categories = ["metalsmith", "dev", "javascript"]
date = 2014-06-06T15:11:36Z
description = ""
draft = false
slug = "implementing-a-metalsmith-plugin"
tags = ["metalsmith", "dev", "javascript"]
title = "Exploring Metalsmith"

+++

*Update - 10th June 14*: *I've extracted the JSON parsing features from the following code into a metalsmith plugin, metalsmith-json, whcih is now available on [github](https://github.com/antiBaconMachine/metalsmith-json) and [npm](https://www.npmjs.org/package/metalsmith-json)*

A few weeks ago, I read about [Metalsmith](http://metalsmith.io), a neat looking static site generator. More recently I was looking at ways of improving an internal static site we maintain as a gateway to download and install mobile artifacts. It seemed like a good opportunity to play with Metalsmith and I'd like to talk through my experience.

#### The problem

The mobile deploy site needs to allow users to download artifacts and display build time parameters. The parameters are all captured in a `config.json` file which is output from every build along with the artifacts. Not every platform is built every build so we need to selectively display download links for each artifact only when available.

#### Structuring the source

We start by laying out our project as follows

    deploy
    	index.js
        src
            index.html
            builds
                1
                    artifact.plist
                    artifact.ipa
                    artifact.apk
                    config.json
                n
                   ...
        templates
            index.hbt
                   
By convention Metalsmith uses a `src` directory for sources and I see no reason to quibble. I'm using the javascript interface to metalsmith, the entry point for which is `index.js`. In the final static site we'll need an `index.html` file to list all the available builds and a numbered directory for each build, containing the config and artifacts for the build.

#### Off the rack plugins

Metalsmith has all the plugins you need to generate a markdown blog, but I wasn't sure how far I'd get with this project before I'd need to start writing my own.

I started with this basic no-op scaffold:

index.js
```language-javascript
var Metalsmith = require('metalsmith');

Metalsmith(__dirname)
    .destination('output')
    .build();
```
        
Running this with `node index.js` will copy all the files from the src dir to the destination 'output'. Not very exciting. 

At this point it's worth summarising the additional steps we need to achieve our goal:

* Parse config.json files to make their properties available to downstream plugins
* Establish which platform artifacts are available
* Generate an entry in `index.html` for each build containing some summary information and links to download artifacts

Looking through the stock plugin list it seems like we might make use of the collections plugin to group all the build config.json files. The template plugin also looks useful for generating the actual content. Nothing further jumps out for the JSON parsing or artifacts. Setting aside these issues for now lets do what we can with off the rack plugins:

```language-bash
$ npm install --save-dev metalsmith metalsmith-collections metalsmith-templates handlebars lodash`
```

index.js
```language-javascript
var Metalsmith = require('metalsmith');
var collections = require('metalsmith-collections');
var templates = require('metalsmith-templates');
var _ = require('lodash');
   
Metalsmith(__dirname)
    .destination('output')
    .use(collections({
        builds: {
            pattern: 'builds/*/config.json'
        }
    }))
    .use(templates('handlebars'))
    .build(function(err, files) {
        console.log(files);
        if (err) throw err;
    });
```   
src/index.html
```
---
title: Home
template: index.hbt
---
```
templates/index.hbt
```language-handlebars
<!DOCTYPE html>
<html>
    {{#each collections.builds}}
        <div class='job'>
            <h2>{{contents}}</h2>
        </div>
    {{/each}}
</html>
```

This gets us an `index.html` which lists the content of all the config.json files. We achieved this by telling metalsmith to tag all `config.json` files in a build directory as part of the global `builds` collection. We then instructed the `index.html` to be transformed using the `index.js` template by placing a template property in it's YAML front matter. The template then traverses the `builds` collection printing the content of each JSON file. 

Also of note is the debug helper function passed to the `build()` function. This will output the final processed fileset to the console and display any errors. This will help catch errors in pipeline whilst we're building it up.

#### Custom plugins

Not bad, but what we really want is for the templates to be able to access the properties of the JSON files. For this we need to parse the JSON before the templates do their thing.

There didn't seem to be much info on implementing plugins on the metalsmith site, but as it turns out a quick look at some existing plugins revealed the  api. 
    
index.js
```language-javascript   
//...
.use(function (files, metalsmith, done) {
    _.each(files, function (file, key) {
        if (path.extname(key) === '.JSON') {
            file.config = JSON.parse(file.contents);
        }
    });
    done();
})
//...
```

So a plugin is just a function which takes 3 args, the fileset, metalsmith instance and a done callback. We iterate over the files and parse any JSON files we find. The JSON properties are made available under the key `config` on the file object where they can be accessed by downstream plugins. This is not especially reusable in this form but its a great way to test out features  before pulling them into configurable reusable plugins.

For artifact resolution we again iterate over all files, this time checking for existence of the artifacts. We could do this in the same iterator as before, but as we're looking to extract these into reusable plugins we keep them separate. For the sake of brevity I won't reproduce this in full here, it's very similar to the plugin above, feel free to jump to the [source](https://github.com/antiBaconMachine/exploring-metalsmith-code) to see it in action.


#### Putting it together

We now have all the ingredients. Here's an updated `index.hbt` which uses the config properties, and the artifact paths.

```language-handlebars
<!DOCTYPE html>
<html>
    {{#each collections.builds}}
        <div class='build'>
            <h2>Build #{{config.build}}</h2>
            <p>version: {{config.version}}</p>
            <ul>
                {{#if platforms.ios}}
                    <li><a href="{{platforms.ios}}">iOS</a></li>
                {{/if}}
                {{#if platforms.android}}
                    <li><a href="{{platforms.android}}">android</a></li>
                {{/if}}
            </ul>
        </div>
    {{/each}}
</html>
```

#### Conclusion

Whilst Metalsmith didn't have all the tools I needed for this project, it was  really simple to write some custom plugins once I figured out how they worked. I'd like to see this better documented on the Metalsmith site. I like the design of the system, it was fun to use and most importantly it saved me time.

The next steps for me are to encapsulate the custom plugins and experiment with the cli interface. 

full source for this experiment is [here](https://github.com/antiBaconMachine/exploring-metalsmith-code).
