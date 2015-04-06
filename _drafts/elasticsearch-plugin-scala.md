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

### Building elasticsearch plugin with Gradle

Elasticsearch plugin is zip file which contains jar file with main plugin class and all dependencies. Another option is just `_site` folder with website content (see [bigdesk pluging repository][bigdesk] as example). 

Basic build process is the following: compile plugin code, run tests and archive with all necessary depepndencies.

You can create Gradle project in IntelliJ or just create `build.gradle` file. 
Gradle build script with annotations: 
```groovy
apply plugin: 'scala' /* to build scala code. 
    if you have mixed project with scala and java,
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

### Elasticsearch Hello world

1. Create plugin main class extended from `AbstractPlugin`, define name (line 7) and description (line 9) for your plugin. Since we want to build REST endpoint we have to import `org.elasticsearch.rest._` and add method `onModule(module:RestModule):Unit` (line 11). Elasticsearch dependency injection is based on Google's DI framework [Guice][guice], it will call this method and pass `RestModule` instance, so you can register your class which containes definition of REST action.

```scala
package hw.elasticsearch

import org.elasticsearch.plugins.AbstractPlugin
import org.elasticsearch.rest._

class HelloWorldPlugin extends AbstractPlugin {
  override def name(): String = "hw-plugin"

  override def description(): String = "Hello World plugin"

  def onModule(module: RestModule): Unit = {
    module.addRestAction(classOf[HWAction])
  }
}
```

2. Create `HWAction.scala` file with class which inherited from `BaseRestHandler`. Annotation `@Inject` tells DI container to inject appropriate dependencies (line 9). We will define the same arguments we have to pass to base class constructor. Scala's primary contractor which is besically body of the class looks pretty laconic and beautiful, doesn't it? 

3. We just need to register this class as a handler. We will use `_` before the url to avoid possible conflicts with usage `hello` as index name. So the next line after class definition is `controller.registerHandler(GET, "/_hello", this)` (line 10).

4. `HWAction` class is not going to be abstract, so we have to implement `handleRequest(RestRequest, RestChannel, Client):Unit` (line 11). Using `RestRequest` we obtain `name` query parameter, execute `answer` method and send response to `RestChannel`. `answer(String):String` method is implemented using awesome pattern matching. 
```scala
package hw.elasticsearch

import org.elasticsearch.client.Client
import org.elasticsearch.common.inject.Inject
import org.elasticsearch.common.settings.Settings
import org.elasticsearch.rest.RestRequest.Method._
import org.elasticsearch.rest._

class HWAction @Inject() (settings: Settings, controller: RestController, client:Client) extends BaseRestHandler(settings, controller, client) {
  controller.registerHandler(GET, "/_hello", this)

  def handleRequest(request: RestRequest, channel: RestChannel, client: Client): Unit =
    channel.sendResponse(new BytesRestResponse(RestStatus.OK, answer(request.param("name"))))

  private def answer(who: String) = Option(who) match {
    case Some("Robert") => "Your Grace!"
    case None | Some("") => "I don't talk to strangers."
    case _ => "Hello, " + who + "!"
  }
}
```
5. **The most important step** to make your plugin visible to elasticsearch is to add `es-plugin.properties` file to the resources directory with the following content:
```properties
plugin=hw.elasticsearch.HelloWorldPlugin
```
If you forget to do so, even after successful installation of the plugin, elasticsearch will ignore your plugin.

### Debugging elasticsearch plugin

We already have elasticsearch dependency in our project with full functional elasticsearch node. At the matter of fact one of the options to connect to elasticsearch cluster using Java API is to start embedded node instance inside your application. Thus debugging of your plugin is very easy. You  just need to add run configuration with main class `org.elasticsearch.bootstrap.ElasticsearchF`.

### Moving forward. Add new script language support.



### Release on github

[4sq]: https://github.com/foursquare/es-scorer-plugin
[idea]: https://www.jetbrains.com/idea/
[why-gradle]: https://google.com/?q=why+gradle
[mvn2gradle]: http://maruhgar.blogspot.nl/2010/12/converting-maven-project-to-gradle.html
[scala]: http://www.scala-lang.org/download/
[gradle]: https://gradle.org/
[es]: https://www.elastic.co/downloads/elasticsearch
[bigdesk]: https://github.com/lukas-vlcek/bigdesk
[guice]: https://github.com/google/guice
