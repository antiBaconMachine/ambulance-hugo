+++
author = "Ollie Edwards"
date = 2014-07-24T09:56:27Z
description = ""
draft = false
slug = "setting-publicpath-for-webpack-dev-server-in-api-mode"
title = "Setting publicPath for webpack-dev-server in API mode"

+++

Recently I had an issue whereby running `webpack-dev-server --content-base www` from the command line was working great, but as soon as I tried to automate it as a gulp task my bundle seemed to never reload properly. 

```language-javascript
new WebpackDevServer(webpack(webpackConfig), { 
    contentBase: "www" 
}).listen(8080, 'localhost', noop);
```
 
Functionally I had assumed that this would be equivalent to the previous command line version. I could see the server was correctly detecting changes and reloading the page but the content never changed.  
 
A quick look at the cli bin revealed that the cli automatically sets the `publicPath` option based on the webpack config `output.publicPath` option. 

```language-javascript
options.publicPath = wpOpt.output && wpOpt.output.publicPath || ""; 
```
     
So I just had to manually add this option to the webpack-dev-server config. 
 
This copying of the webpack publicPath seems like a sensible default to me, which begs the question, **why isn't this the default for the API?**
 