---
layout: post
title: Releasing to maven central
---


Ever wanted to be a 'superstar' and have your project jars available to the entire world from something like [maven central](http://search.maven.org/) ? 
I did, for a small little [project](https://github.com/mihaicostin/hibernate-l2-memcached), 
and it turns out it’s not impossible, although there are more than a couple of steps that need to be taken to get there.

Initially, I’ve opted for [jitpack](https://jitpack.io/) to make the jar available to (my) other projects, and that works quite nice.
Kudos to jitpack, by the way, for making that so easy.
However, that extra repository in my other project was nagging me

```xml
<repository>
    <id>jitpack.io</id>
    <url>https://jitpack.io</url>
</repository>
```

So I decided to bite the bullet and get that initial project to maven central. How hard could it be?

First thing that I had to do was read the available [documentation](http://central.sonatype.org/pages/apache-maven.html). 
It’s well organized (with videos and samples), but it takes quite a while to get through everything.

Then, I went ahead and create a new user on [sonatype](https://issues.sonatype.org/secure/Signup!default.jspa), 
followed by a new ticket for my project, 
just as described [here](http://central.sonatype.org/pages/ossrh-guide.html#create-a-ticket-with-sonatype).
If the project is hosted on github (as it is in my case) the group id and artifact id should follow the github url. 
Make sure to write down the username and password for the sonatype account ([lastpass](https://lastpass.com/) anyone?)
as you will need them later on in your local `.m2/settings.xml` config.

And this was the [ticket](https://issues.sonatype.org/browse/OSSRH-17754) that I’ve made for that project.

As they state in the documentation, these initial steps might take up to a couple of days but in my case it was pretty fast, so don’t be deterred by that.

Before you can publish anything to sonatype you will need to check the [requirements](http://central.sonatype.org/pages/requirements.html) and provide:

- Sources 
- Javadocs 
- Signed artifacts

There is already a sample pom provided in their documentation or you can take a look at the one that I made for my project over [here](https://github.com/mihaicostin/hibernate-l2-memcached/blob/master/pom.xml).</br> 
In my case, the release profile takes care of generating the sources, javadoc jar and sign everything.</br> 
You will also need the pom sections licenses, developers, scm, distributionManagement and the nexus-staging-maven-plugin (v 1.6.3) configured… 
so pretty much everything from that pom file.</br> 
Notice that I’ve chosen to set the **autoReleaseAfterClose** flag for the staging plugin to false so that I can manually review the artifacts (on the sonatype site) before the actual release. 
You can read more about it [here](http://books.sonatype.com/nexus-book/reference/staging-sect-intro.html).

In order to sign the jars you will (obviously) need a tool that can sign them along with a pair of keys.
For macos you can use gnupg (`brew install gnupg` , for brew users).</br>
You will then need then to generate a key pair (`gpg --gen-key`), 
publish your keys (and make sure to check you can retrieve the key after a couple of minutes) and then, finally, update your local `.m2/settings.xml`.
You can read more about it [here](http://central.sonatype.org/pages/working-with-pgp-signatures.html) 

Here’s how my local settings.xml file looks like:

```xml
<localRepository/>
<interactiveMode/>
<usePluginRegistry/>
<offline/>
<pluginGroups/>
<servers>
    <server>
        <id>gpg.passphrase</id>
        <passphrase>your-gpg-key-password</passphrase>
    </server>
    <server>
        <id>ossrh</id>
        <username>your-sonatype-user</username>
        <password>your-sonatype-password</password>
    </server>
</servers>
<mirrors/>
<proxies/>
<profiles/>
<activeProfiles/>
```

You are almost done… the only thing left to do is to actually release a new version of the project.</br>
Considering that the current “dev” version is 1.0.0-SNAPSHOT, you will need to ditch the “snapshot” (manually or by using a maven release plugin), turning it into 1.0.0. 
Then commit & push the changes. 

If you’re using a release profile, like I did, you can now issue the command 

`mvn clean deploy -P release` 

and enjoy the text scrolling in your console. 


If you have the autoReleaseAfterClose flag set to false (like I do), you will also need to do a manual release of your artifacts. 
This means:

- Log in to [https://oss.sonatype.org](https://oss.sonatype.org)
- Go to staging repositories 

![Staging repositories menu](/images/2016-09-09-maven/sonatype-staging.png) 

- Search for your repository 
- Release

![Release](/images/2016-09-09-maven/sonatype-release.png)

and you're done.
