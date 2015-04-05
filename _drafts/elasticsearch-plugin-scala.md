---
layout: post
title: "Writing your own elasticsearch plugin in Scala assembled by Gradle step-by-step"
description: "Elasticsearch is quite popular search engine, Scala is quite popular JVM language and Gradle is quite popular build tool. Let's try all of them at once." 
category: dev
tags: [ Scala, elasticsearch, Gradle ]
---
{% include JB/setup %}

According to wiki
> Elasticsearch is a search server based on Lucene. It provides a distributed, multitenant-capable full-text search engine with a RESTful web interface and schema-free JSON documents.

After more then one year of experience with elasticsearch, I can say it does work very well. Elasticsearch developers made very good job not only to develop high quality search database but also to keep it out of new features which are not necessary and be focused on critical and generic functionality. At the same time they provide very good plugin system to provide the way to extend elasticsearch for you needs when basic functionality is not enough for you. You can find a lot of different plugins on Github which monitor elasticsearch cluster, collect data from different sources like Twitter, RabbitMQ, MongoDB (so called "rivers"), Forsquare [published on github][4sq] plugin for custom geo-based scoring (written in Scala by the way) and many more. 

### Install Environment

I will use [IntelliJ IDE][idea] Community edition. It has built in support for Gradle. If you are not familliar with Gradle yet and don't know why you should spend time on it, just read top 3 from Google search "[Why Gradle][why-gradle]". I like it for most of its advantages, but I love it for flexibility given by Groovy (you can write code in your build script like with Rake for Ruby or FAKE for .NET) and easy to read syntax in comparison to old XML based build tools.
We are going to develop elasticsearch plugin in [Scala][scala]. The reason why I didn't choose SBT is usually we have already Java project (and got lucky to have Gradle as a build tool. if not, you may consider [migration from Maven][mvn2gradle]). Your team is suttisfied to use Java for mainstream development, but new plugin is more likely will be easier to write in functional style, because you need to do a custom text analysis or processing, and functional languages are proved suitable tool for that. You are most likely don't think even to change build tool just for ~3% of you codebase. With Gradle it's very easy to build your plugin in Scala.
So make sure you have [Scala][scala], [Gradle][gradle] and [elasticsearch][es] installed and of course Java installed. I will use Java 8.

### Elasticsearch Hello world

### Moving forward. Add new script language support.

### Debugging elasticsearch plugin

### Building elasticsearch plugin with Gradle

Elasticsearch plugin is zip file which contains jar file with main plugin class and all dependencies. Another option is just `_site` folder with website content (see [bigdesk pluging repository][bigdesk] as example). 

Basic build process is the following: compile plugin code, run tests and archive with all necessary depepndencies.

You can create Gradle project in IntelliJ or just create `build.gradle` file. 
Gradle build script with annotations: 

```groovy
apply plugin: 'scala' 

/* to build scala code. if you have mixed project with scala and java,
you can add second line with java instead of scala */

sourceCompatibility = 1.8
version = '1.0'

repositories {
    mavenCentral()
    mavenLocal()
}

// additional configuration to tag dependencies to be archived with plugin jar
configurations {
    includeJars
}

dependencies {
    compile 'org.scala-lang:scala-library:2.11.4'
    compile 'org.elasticsearch:elasticsearch:1.4.4'
    testCompile 'junit:junit:4.11'
    includeJars 'org.scala-lang:scala-library:2.11.4' // include this dependency
}

// task to archive plugin jars
task buildPluginZip(type: Zip, dependsOn:[':jar']) {
    baseName = 'hw-plugin'
    classifier = 'plugin'
    from files(libsDir) // include output dirictory into archive
    from { configurations.includeJars.collect { it } } // include dependencies to archive
}

// define artifacts
artifacts {
    archives buildPluginZip
}
```

Run Gradle build with the following console command 
```
gradle build buildPluginZip
```
As the result we will have zip file ready to install to elasticsearch 

```
bin\plugin hw-plugin -install hw-plugin -url=file:/path_to_zip/hw-plugin.zip
```

### Release on github

[4sq]: https://github.com/foursquare/es-scorer-plugin
[idea]: https://www.jetbrains.com/idea/
[why-gradle]: https://google.com/?q=why+gradle
[mvn2gradle]: http://maruhgar.blogspot.nl/2010/12/converting-maven-project-to-gradle.html
[scala]: http://www.scala-lang.org/download/
[gradle]: https://gradle.org/
[es]: https://www.elastic.co/downloads/elasticsearch
[bigdesk]: https://github.com/lukas-vlcek/bigdesk
