+++
author = "Ollie Edwards"
date = 2014-12-23T12:44:42Z
description = ""
draft = false
slug = "unit-testing-ios-code-with-swift-and-manual-mocks"
title = "Unit testing iOS code with Swift and manual stubs"

+++

In general I find unit testing on the iOS platform to be a pain. Xcode's built in test library, XCTest, is very basic and whilst I'm reasonably inexperienced with iOS development, it does seem the community as a whole doesn't really rate unit testing. Automated front end integration testing seems to have a much higher mind share.  
 
Now I'm not against this approach at all. As with any front end tech, there are limits to what isolated unit testing can achieve. An application is about more than the sum of its parts and that simply can't be expressed with unit tests. However, I still feel that a solid set of unit tests for model objects and even some controller logic is extremely beneficial. 
 
A great unit test suite should be like readable documentation, and that's why I've been writing my iOS tests in Swift, even though the app itself is written in Objective C. Whilst readability is a matter of opinion, I don't think you'll find many who rate Objective C higher in that respect. Certainly in our primarily JavaScript focused team, we find it much easier to scan. 
 
Swift is still very new and as such there are a few areas lacking, one of which is metaprogramming support, which of course means automated mocking and stubbing for unit tests is currently impossible. 
 
I'd like to share my first attempt at a manual inline stubbing in Swift. 
 
First we define a simple captor object to track function invocations:
```language-swift
private class FunctionCall {
    let name:String
    let args:[Any]?
     
    init(name:String, args:[Any]?) {
        self.name = name
        self.args = args
    }
}    
```

We then override our target class with a few extra helpers. In this case I'm stubbing a pebble watch model object.

```language-swift
 private class MockPebbleDevice: PebbleDevice {
    //Track the calls made
    var calls:[FunctionCall] = []
        
    //We can manipulate the results of each function call as needed
    var responses:[String:AnyObject] = [
        "isConnected":false
    ]

    //helper which logs each function and returns appropriate result
    func stub(functionName:String, args:[Any]?) -> AnyObject? {
        calls.append(FunctionCall(name: functionName, args: args))
        return responses[functionName]
    }
        
    //stub functions follow here
    override func isConnected() -> Bool {
        return stub("isConnected", args: nil) as Bool
    }
        
    //more stubs as required
    ...
        
}
```
    
We can then inject this model to our fixture, and control the results of each function call by manipulating the responses dictionary. Here's an extract from one of the tests using this structure:

```language-swift
...
fixture.didReceiveWatchSecret(VALID_SECRET, withCallback: nil)
        
XCTAssertEqual(self.device.calls.count, 3, "We have made 3 calls to the watch")
        
let nameCall = self.device.calls[0];
XCTAssertEqual(nameCall.name, "name", "First call was to get device name")
let connectedCall = self.device.calls[1];
XCTAssertEqual(connectedCall.name, "isConnected", "We checked connectivity as the 2nd call")
...
```
    
The process of setting up the mock object is very manual, but it got us where we needed to be for these test cases. 

I'd love to hear about your experiences tackling the problem of mocks and stubs in Swift. One of the exciting things about working with a new language is that no one yet knows the best answers to these problems and I'm sure there are many other interesting approaches out there.
    

