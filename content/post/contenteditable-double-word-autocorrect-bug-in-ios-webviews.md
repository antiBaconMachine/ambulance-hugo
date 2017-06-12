+++
author = "Ollie Edwards"
date = 2015-04-29T10:42:35Z
description = ""
draft = false
slug = "contenteditable-double-word-autocorrect-bug-in-ios-webviews"
title = "Contenteditable double word autocorrect bug in iOS webviews"

+++

Came accross a very frustrating bug this week, it looks like this:

1. Within an ios webview start typing in a content editable div.
2. Make an obvious typo so that autocorrect makes a suggestion

	![](/content/images/2015/Apr/IMG_0026-2.PNG)
3. Tab a button or link somehwere else on the page.
4. Autocorrect will insert two copies of the suggested word.

	![](/content/images/2015/Apr/IMG_0027.PNG)

I've not seen this detailed anywhere else so I have provided a concrete [example](https://github.com/antiBaconMachine/contenteditableIosBug) cordova app.

As yet I haven't found any reasonable mitigation. Ideas?
