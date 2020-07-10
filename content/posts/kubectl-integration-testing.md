+++
title = "Integration testing kubectl or other external binaries"
subtitle = "Fake it like you mean it"
date = 2020-07-10T15:04:24Z
tags = ['kubernetes', 'shell', 'testing']
draft = false
+++

I found myself needing to write an integration test for a complex workflow invovling dynamic payloads being delivered to `kubectl`. The idea is that a user can call a rest endpoint, which will kick off a whole Rube Goldberg machine, the end result is that a kubernetes manifest is generated and applied. There are many moving parts and a solid tsting stratergy is needed. In particular it's essential that we have full end to end testing that shows a user can reliably submit their payload and a real job is spawned on a cluster as a result.

End to end tests are the closest automated tests to an actual user experience so arguably the most reliable in terms of guaging the actual experience of a user. They are also typically the slowest tests both in terms of actual run time and the lag time between submitting code and seeing a test result. I wouldn't dream of undertaking this project without the ATs watching my back but I also crave the fast feedback loop of an integration test that will run fast right in my CI pipeline. The requirements for my test suite are:

* It runs in our existing CI environment which is github actions
* It's runtime impact is measured in seconds not minutes so that it can run on every commit

The systems inputs are HTTP post bodies in JSON format. The outputs are kubernetes jobs running on a cluster. It's unlikely I'd be able to bootstrap a whole local kubernetes cluster, deploy several microservices and run the test suite in <1 minute so it feels quite natural to move the boundary a bit so we say the inputs are HTTP json payloads and the outputs are templated kubernetes manifest in yaml format. We can then make assertions on the output jobs without doing the slow heavy work of actually applying them to a cluster.

The way I chose to implement this boundary change was by replacing the kubectl binary itself with a fake. As the CI environment is github actions we're already targetting a docker runtime so it also felt natural to do the binary replacement using a docker volume mount.

As we're very interested in running assertions on the input to kubectl what we want is a kubectl fake that essentially just passes through it's `stdin` to `stdout` e.g. `cat`. Throwing together a one line shell script that simply calls `cat` is actually a pretty much all we need for this form of testing, it's fast and gives us everything we need to run our assertions.

However under certain conditions the command we use with kubectl is `delete` not `apply` which does not expect stdin. This means that cat will hang forever. We could code up a special case for `delete` in our fake to handle this but I really wanted the test fake to be as dumb as possible, we don't want to get into a state where the logic in the test fake is sufficiently complex to require testing.

I couldn't come up with a _good_ way of writing this to be honest so I fell back on everyone's favourite testing get of jail card, a timeout. My first implementation used the `timeout` command.

```
#! /usr/bin/env ash
set -x
# if we get some data on stdin within a second then pass it straight through, otherwise just exit quietly
if ! timeout 1 cat; then
  # 124 is the exit code that cat returns on sigterm, which means there was probably no useful stdin
  if [[ $? == 124 ]]; then
    exit 0
  fi
fi
```
Note I've designed this to run in an alpine linux container so it targets the `ash` shell. This should be fine in `bash`, I believe the `-d` option is _not_ posix compliant though so i can't guarentee this will work for every shell.

This solution is _ok_ but it's already quite complex and it waits a whole second which is an eternity. `timeout` doesn't do fractional values so we can't get it lower than that. My second attempt uses the `read` builtin and `echo` instead of `cat`.

```
#! /usr/bin/env ash
set -x
read -t 0.1 -d $'\0' stdin
echo "$stdin"
```

`read` supports fractional timeout values to it's `-t` flag and we don't need to handle any non zero exit code for the timeout case. If we change the delimiter to the null character then we get the cat like behavior of exiting after stdin is closed.

This is much closer to the trivial one liner I originally envisaged and there's now only 100ms of ead time per test that uses `delete`. It would definitely be faster to code for the delete case specifically but I like this light touch script and in practice i don't _yet_ notice the penalty as there are only two big integration tests. If we flesh out the suite more over time then I'll probably code up something.

I like this approach of testing external binaries soley based on their input, or put another way our systems output. It's a great comporimse and gets us feedback much earlier than waiting for the full AT report. You could use the same approach to isolate and test any expensive binary calls _where you are not normally interested in their output_. Let know what you think of this approach or any other approaches for isolting binaries for testing purposes.
