+++
author = "Ollie Edwards"
date = 2015-09-10T14:36:33Z
description = ""
draft = false
slug = "retrieving-maven-snapshots-from-nexus-with-gradle"
title = "Retrieving asset snapshots from Nexus with Gradle"

+++

I've recently been building a Tizen wearable widget to be packaged with one of our Android apps. This is achieved by bundling the built widget in the assets dir of the APK. 

The simplest way to develop in this way is to symlink the widget to the right place. Once this goes out for testing though you're either going to have to move the binary to source control and commit to keeping it up to date, or have your dependency manager handle this for you.

I prefer the second option so I set about managing the widget with our local Nexus install. For dumb binary artifacts with no special significance to the build system we use the POM packaging type in Nexus. You can do this either from the nexus web interface or via the mvn publish plugin. I cheated and used the [nexus-deployer](https://www.npmjs.com/package/nexus-deployer) as a gulp task which was very simple. The only slightly interesting thing I did here was infer whether we have a SNAPSHOT or a release based on whether the output of git describe (Which I have lying around in the gitVersion variable) and publish the the correct repo.

```language="javascript"
   var isSnapShot = version.indexOf('-') != -1;
   var v = gitVersion.match(/v(([0-9]*\.){2}[0-9]*)/)[1];
   var config =  {
            groupId: CONFIG.nexus.group,
            artifactId: CONFIG.projectName,
            version: isSnapShot ? '1.0-SNAPSHOT' : v,
            packaging: 'wgt',
            auth: {
                username:'***',
                password:'***'
            },
            pomDir: '.buildResult/pom',
            url: isSnapShot ? CONFIG.nexus.snapshotRepo : CONFIG.nexus.releaseRepo,
            artifact: "myWidget.wgt",
            quiet: false,
            insecure: true
        }
```

In the Gradle project the first thing we need to do is set up the correct repos.

```language="groovy"
	allprojects {
    repositories {
        jcenter()
        maven {
            url 'http:////<NEXUS_URL>/nexus/content/repositories/releases/'
        }
        maven {
            url 'http:////<NEXUS_URL>/nexus/content/repositories/snapshots/'
        }
    }
}
```

We then need a way to determine whether to use the release or snapshot for a given build. Gradle doesn't recognise Maven SNAPSHOT artifacts by default, which is fair enough given it does not position itself as Maven compatible. We therefore need to implement our own switching logic. I thought about using the android build variants 'debug' and 'release' as the key for this switch, but I decided to go for complete flexibility and implement this as a gradle project property.

```language="groovy"

//1
configurations {
    tizen
}

//2
configurations.all {
	//Check for updated 'changing' artifacts every build
    resolutionStrategy.cacheChangingModulesFor 0, 'seconds'
}

dependencies {
	//3
    ext.tizenVersion = tizenSnapshot.toBoolean() ? "1.0-SNAPSHOT" : "0.+"
    tizen ("com.mobbu.tizen:passwear:$tizenVersion@wgt") {
        changing = true
    }
}

def assetsDir = new File(projectDir, 'src/main/assets')

//4
task tizenClean(type: Delete) {
    delete fileTree("${assetsDir}") {
        include '**/*.wgt'
    }
}

task tizenCopy(type: Copy, dependsOn: tizenClean) {
    from { // use of closure defers evaluation until execution time
        configurations.tizen
    }
    into "${assetsDir}/"
}

//5
preBuild.dependsOn tizenCopy
```

Lots to note here:

1. We create a configuration group for the tizen artifact(s). This is effectively just a label for a group of dependencies e.g. 'compile'. The reason for this will become clear in (4).
2. The closest approximation of a SNAPSHOT in Gradle is a dependency which is marked as 'changing'. We prevent all such marked dependencies from being cached here as, in this case, these are fast and arbitrarily changing dev snapshots, not nightlies.
3. We base the snapshot/release flag on a project property 'tizenSnapshot' Gradle properties are really cool as they can be set in a multitude of different ways. By default I've set the local gradle.properties file to have tizenSnapshot=false but this can be overidden, by command line, global gradle.properties or env var.
4. Now we have retrieved the asset it needs to be put in the right place, so we clean any previous widgets and copy the newly retrieved one in. This shows the utility of our custom configuration as we now have a handle to grab the required assets with.
5. We add the clean/copy chain as a dependency to preBuild.

This approach could easily be extended to a more general 'assets' configuration. It was reasonably fast to throw together and gets the job done.

I do find with gradle I have to write a lot of these kind of scripts to get real work done whereas other systems have common well defined paradigms which generally get you there. It does at least make it fun and easy to write this kind of thing when required and most importantly it doesn't make me write build logic in XML.