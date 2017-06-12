+++
author = "Ollie Edwards"
date = 2015-01-21T13:28:24Z
description = ""
draft = false
slug = "ios-code-signing-a-tragicomedy-in-3-parts"
title = "iOS Code Signing: A Tragicomedy In n Parts"

+++

There's more than enough code signing invective washing around the internet already, so instead of moaning about all the ways the process is confusing, I'll start with a compliment: for new solo developers, xcode typically just works for signing, I think Apple have done a decent job. Unfortunately if you need to run a CI job for your app or otherwise sign from the command line or use multiple certificates and profiles, things get tricky.

###Background

I think this graph is the best expression of what happened to me this week.

![](/content/images/2015/Jan/iosCodeSigningComplexity.png)

When you're a new iOS developer code signing is weird and scary, but you can do enough painting by numbers to get you through it with only mild annoyance. If you are a lone developer with one organisation then this might be enough, and you can continue to develop largely in ignorance. If you should ever need a second account, and therefore a second set of certificates, well you're going to need to grow up fast. 

Historically we've used an enterprise account to do all app distribution as we've only had a need for in house apps. This is really convenient as the app just works on any device.

We're now midway through our first actual submission to the app store. This seemingly required us to open a completely different organisation account with Apple. Here's where the fun begins.

###Name your organisations appropriately

This may seem obvious but honestly, one major issue we faced here is that for some reason our new shiny organisation account has exactly the same name as our enterprise account. Given how much paperwork and bureaucracy we had to fight through to get the enterprise account together in the first place I don't see it being easy to resolve. Clearly some of the blame lies on us here, but seriously, company name is not a unique key in iTunes connect database?

There are some cosmetic issues with having two identical names. For instance whenever we are asked by xcode or the developer portal to select which account to use from a drop down box, the two options are identical. For me I just know that the top option is the org and the bottom option is the enterprise but out Jenkins box is reversed. This is a pain, but I can live with it.

Distribution certificates are named after the organisation they are attached to however, so if you want certificates for both in the same keychain then these are also hard to distinguish. If you leave xcode to its own devices, you might get away with it, but if you try to sign on the command line you'll likely get an ambiguous certificate error.

A straight forward workaround to this is to specify the certificate by SHA fingerprint so

```language-bash
xcrun --sdk iphoneos PackageApplication --sign <SHA fingerprint> ...
```

You can get the fingerprint by inspecting the appropriate certificate in your keychain.

###Signing and resigning

A typical strategy for building on the command line is to have xcode build the app with whatever certificate it likes, then resign it on the command line when we make the ipa. This will work well enough when you only have one appropriate provision profile. 

The issue I ran into was that xcode was automatically choosing a provision profile in the initial build, then we were attempting to replace this provision profile later. There's nothing in particular wrong with this but xcode does some build magic using the provision profile, such as setting up your entitlements plist.

In this case xcode was erroneously choosing a development profile intended for an app from our other organisation which caused an entitlements mismatch once we replaced the profile. Worse, this is not a build error, you'll only spot the issue when you come to install.

This is easily resolved by specifying the correct provision profile in xcode. However life this week just hasn't been that simple. The particular app in question is a cordova app, and as part of the build process we blat the xcode project and build it from scratch.

I therefore had to hack in an ```after_prepare``` build hook to add the correct provision profile to the build.xcconfig script like so:

```language-javascript
#!/usr/bin/env node

var fs = require('fs');
var path = require('path');
var rootdir = process.argv[2];

if (rootdir) {
    var xcconfig = "platforms/ios/cordova/build.xcconfig",
        fullfilename = path.join(rootdir, xcconfig);

    if (fs.existsSync(fullfilename)) {
        var prov = "\nPROVISIONING_PROFILE=11111111-1111-1111-1111-111111111111";
        fs.appendFileSync(fullfilename, prov);
    } else {
        console.error("missing: "+fullfilename);
    }
} else {
    console.error("Could not find dir");
}
```

Again the key is to specify the signing asset you need via hash rather than name.

###Enterprise expiry

But wait, we still weren't done.

I had now fixed everything up so that two products could be signed with certificates from the same keychain, with the same name, but different apple organisations. I was then informed that some old enterprise artifacts would no longer install. I tested and was able to install them just fine. I tried a few different phones and eventually found one on which installation failed consistently.

I checked the log and saw that the artifact had expired several weeks ago, which blew my mind as I had just installed the app successfully not five minutes ago.

After scratching my head and pouring over documentation for a few minutes I figured out that the devices which were allowing the expired artifact where all devices which had previously had a more recent version of the app with a refreshed provision profile installed on them. When you delete an app on iOS it leaves it's profile behind, and this profile can override older versions of the same profile. This is a handy feature as you can refresh your enterprise apps by just installing new profile.

In this case, the solution to this issue was to simply repackage the old artifact with an up to date provision profile, and as the certificate still has  almost two years to run, we'll be able to do so one more time when the new profile expires after it's year term.

###Conclusion

It's been a break neck couple of days piecing all this together, but I do now know a great deal more about iOS provisioning and the platform in general. We have to take our fair share of the blame for the issues as our build process is definitely not standard and our situation just isn't typical for iOS development. There isn't a great deal of point in bleating about Apple forcing us to do things their way or the highway, but it would be nice if they could document their way just a teeny bit better.
