+++
author = "Ollie Edwards"
date = 2015-02-25T19:38:10Z
description = ""
draft = false
slug = "objective-c-to-swift-case-sensitivity-gotcha-with-external-parameter-names"
title = "Objective C to Swift case sensitivity gotcha with external parameter names"

+++

I hit a slight edge case on a project interopping between Objective C and Swift the other day. The app itself is all Objective C with the test suite in Swift. In this case I was implementing an Objective C delegate protocol in Swift as a test. 

I've distilled the issue down into a simple [demo project](https://github.com/antiBaconMachine/caseSensistiveSelectorTest), so I'll be using code from that to describe the problem. We start with an Objective C protocol:

```language=C
@protocol fooProtocol<NSObject>
- (void) didBarWithSpam: (NSString*)spam;
- (void) didBarWithSpam: (NSString*)spam andEggs: (NSString*)eggs;
- (void) didBarWithSpam: (NSString*)spam AndSausages: (NSString*)sausages;
@end
```

The key is the 3rd method which has a second argument labelled "AndSausages" with an initial cap. If we implement this in Swift Xcode will auto generate the following method signature for this method:

```language=swift
func didBarWithSpam(spam: String!, andSausages sausages: String!)
```

Note this has been implemented with initial lower case for the label. Changing to the correct initial cap form  results in a compile time error `'does not conform to protocol 'fooProtocol'`. 

Back in Objective C, using the standard pattern of checking if a delegate implements a selector before calling it will now fail due to the differing cases:

```language=c
 if ([self.delegate respondsToSelector:@selector(didBarWithSpam:AndSausages:)]) {
        [self.delegate didBarWithSpam:@"SPAM" AndSausages:@"SAUSAGES"];
    }
```

If you check out the [demo project](https://github.com/antiBaconMachine/caseSensistiveSelectorTest) and run the tests you'll see this failure for yourself.

Now by convention I guess it can be argued that you should have intitial lower case names anyway, but there's no enforced standard on that so it seems a bit out of order that Swift is more restrictive.

Can anyone shed any light on why this is the case?