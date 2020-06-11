+++
author = "Ollie Edwards"
date = 2016-04-07T08:16:20Z
description = ""
draft = false
slug = "nvm-upgrade-to-latest-node"
title = "NVM Upgrade to latest node"

+++

Node moves fast. Like most people I manage the frequent updates with NVM. A feature I wish existed in NVM is a one liner to updgrade to the latest available node and set this as the system default.

Fortunately this is not too hard to script, here's my attempt:

```bash
#! /usr/bin/env bash
source $NVM_DIR/nvm.sh
CURRENT=`nvm current`
LATEST=`nvm ls-remote | tail -n 1 | grep -o "v[0-9\.]*"`

echo "current ($CURRENT), latest ($LATEST)"

nvm install $LATEST --reinstall-packages-from=$CURRENT
nvm alias default $LATEST
```

Reasonably self explanatory but a couple of points are worth noting:

NVM only affects the environment of your current shell so to get the binary on your path you need to source `$NVM_DIR/nvm.sh`

This also means that the script will only change node installation for the shell the script runs in. If you'd like that to be your current termainl then you need to save this script to e.g. `nvm-upgrade.sh` and run it as `source nvm-upgrade.sh`.

To change installation in any other previously running terminal sessions you'll then need to run `nvm use default` to update to the new installation.

As your global npm packaes should come along for the ride thanks to the `--reinstall-packages-from` flag this whole script is fairly non disruptive and means I'm now keeping much more up to date.