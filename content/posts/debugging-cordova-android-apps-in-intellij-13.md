+++
author = "Ollie Edwards"
categories = ["cordova", "intellij", "android"]
date = 2014-07-11T08:53:03Z
description = ""
draft = false
slug = "debugging-cordova-android-apps-in-intellij-13"
tags = ["cordova", "intellij", "android"]
title = "Debugging Cordova Android apps in IntelliJ 13"

+++

Debugging an Android app in IntelliJ is fairly straightforward. It's slightly harder to figure out how to spin up a cordova app. 

First things first we need to define an android module. The only way I've got this to work is by importing a module rather than creating a new one. Open the project structure and go to modules screen. right click your project and select ```add > import module```. In theory the wizard should allow you to set up everything you need. In practice I've found you have to do loads of manual settings.

On the first screen select ```Create module from existing sources``` and continue.

On the sources just leave everything checked, we'll need to manually configure this later anyway.

On the dependencies screen uncheck everything, as there seems to be a bug whereby even if you add the requisite libraries from this screen the resulting paths will be relatively referenced from the wrong location..

Just click through the final 2 screens.

Now go to project structure and select your new module. On the sources tab remove all sources and mark the ```platforms/android/src``` and ```platforms/android/CordovaLib/src``` directories as source roots.

On the paths tab we need to set a valid output dir for built APKs. This has to be an existing dir, it will not get created for you. There see

On the dependencies tab add the ```platforms/android/lib``` folder as a ```folder``` dependency.

At this point you should be able to build your project using ```build > make module {YOUR_MODULE}```

In the file browser locate the APK you just built and copy it's absolute path.
 
Now return to project structure and select the module facet and go to the packaging tab. In the APK path field paste the absolute path to the APK. This field has a drop down file browser but inexplicably it will not allow you to select an `.apk` file.

Now create a new android run configuration. If all has gone well no further setup should be required and debugging on device should 'just work'.