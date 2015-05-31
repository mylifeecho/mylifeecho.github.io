---
layout: post
title: "Elasticsearch, Scala, Gradle. Writing plugin step-by-step"
description: "Elasticsearch is quite popular search engine, Scala is quite popular JVM language and Gradle is quite popular build tool. Let's try all of them at once."
category: dev
tags: [ Scala, elasticsearch, Gradle ]
---
{% include JB/setup %}

According to wiki
> Elasticsearch is a search server based on Lucene. It provides a distributed, multitenant-capable full-text search engine with a RESTful web interface and schema-free JSON documents.

After more then one year of extensive experience with elasticsearch, I can say it does work very well. Elasticsearch developers made very good job not only to develop high quality search database but also to keep it out of new features which are not necessary and be focused on critical and generic functionality. At the same time they provide very good plugin system to extend elasticsearch for you needs when basic functionality is not enough for you. You can find a lot of different plugins on Github which monitor elasticsearch cluster, collect data from different sources like Twitter, RabbitMQ, MongoDB (so called "rivers"), Forsquare [published on github][4sq] plugin for custom geo-based scoring (written in Scala by the way) and many more. I'm going to describe how to build simple hello world plugin in Scala using Gradle, release it on Github and start to use it.

## Install Environment

I will use [IntelliJ IDE][idea] Community edition. It has built in support for Gradle. If you are not familliar with Gradle yet and don't know why you should spend time on it, just read top 3 from Google search "[Why Gradle][why-gradle]". I like it for most of its advantages, but I love it for flexibility given by Groovy (you can code tasks in your build script the same way using Rake for Ruby or FAKE for .NET) and easy to read syntax in comparison to old XML based build tools.
We are going to develop elasticsearch plugin in [Scala][scala]. There is a nice build tool for Scala called SBT. It's definetelly tool to go on pure Scala projects, but usually we have already Java project (and got lucky to have Gradle as a build tool. if not, you may consider [migration from Maven][mvn2gradle]). Your team is suttisfied to use Java for mainstream development, but new plugin is more likely will be easier to write in functional style, because you need to do a custom text analysis or processing, and functional languages are proved suitable tool for that. You are most likely don't think even to change build tool just for ~3% of you codebase. With Gradle it's very easy to build your plugin in Scala.
So make sure you have [Scala][scala], [Gradle][gradle] and [elasticsearch][es] installed and of course Java installed. I will use Java 8.

## Building elasticsearch plugin with Gradle

Elasticsearch plugin is zip file which contains jar file with main plugin class and all dependencies in it. There is another option just `_site` folder with website content (see [bigdesk pluging repository][bigdesk] as example), but we will write real scala plugin which will integrate into elasticsearch and extend its functionality. Basic build process is the following: compile plugin code, run tests and archive with all necessary depepndencies. Resulting archive is a valid plugin distribution.

Create Gradle project in IntelliJ or just create `build.gradle` file.
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
    compile 'org.elasticsearch:elasticsearch:1.5.2' // same the server version of your elasticsearch
    testCompile 'junit:junit:4.11'
    includeJars 'org.scala-lang:scala-library:2.11.4' // include this dependency
}
// task to archive plugin jars
task buildPluginZip(type: Zip, dependsOn:[':jar']) {
    baseName = 'hello-plugin'
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
bin\plugin --install hello-plugin --url=file://path_to_zip/hello-plugin.zip
```

## Elasticsearch Hello plugin

1) Our first step will be to create the main class of the plugin extended from `org.elasticsearch.plugins.AbstractPlugin`, define name (line 7) and description (line 9) for your plugin. Since we want to build REST endpoint we have to import `org.elasticsearch.rest._` and add method `onModule(module:RestModule):Unit` (line 11). Elasticsearch dependency injection is based on Google's DI framework [Guice][guice], it will call this method and pass `RestModule` instance, so you can register your class which containes definition of REST action.

```scala
package hello.elasticsearch

import org.elasticsearch.plugins.AbstractPlugin
import org.elasticsearch.rest._

class HelloPlugin extends AbstractPlugin {
  override def name(): String = "hello-plugin"

  override def description(): String = "Hello plugin"

  def onModule(module: RestModule): Unit = {
    module.addRestAction(classOf[HelloAction])
  }
}
```

2) Create `HelloAction.scala` file with class which inherited from `BaseRestHandler`. Annotation `@Inject` tells DI container to inject appropriate dependencies (line 9). We will define the same arguments we have to pass to base class constructor. Scala's primary contractor which is basically body of the class looks pretty laconic and beautiful, doesn't it?

3) We just need to register this class as a handler. We will use `_` before the url to avoid possible conflicts with usage `hello` as index name. So the next line after class definition is `controller.registerHandler(GET, "/_hello", this)` (line 10).

4) `HelloAction` class is not going to be abstract, so we have to implement `handleRequest(RestRequest, RestChannel, Client):Unit` (line 11). Using `RestRequest` we obtain `name` query parameter, execute `answer` method and send response to `RestChannel`. `answer(String):String` method is implemented using awesome pattern matching.

```scala
package hello.elasticsearch
/* all imports */
class HelloAction @Inject() (settings: Settings, controller: RestController, client:Client)
    extends BaseRestHandler(settings, controller, client) {

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

5) **Probably the most important step** to make your plugin visible to elasticsearch is to add `es-plugin.properties` file to the resources directory with the following content:

```properties
plugin=hello.elasticsearch.HelloPlugin
```

If you forget to do so, even after successful installation of the plugin, elasticsearch will ignore your plugin and you may waste your time trying to figure out what's wrong with your code.

## Debugging elasticsearch plugin

We already have elasticsearch dependency in our project with full functional elasticsearch node. At the matter of fact one of the options to connect to elasticsearch cluster using Java API is to start embedded node instance inside your application. Thus debugging of your plugin is very easy. You just need to add run configuration with main class `org.elasticsearch.bootstrap.ElasticsearchF`. You can use VM options to change elasticsearch configuration. Any option in `elasticsearch.yml` file can be used with `es.` prefix. So for instance if you want to change cluster name to debug your plugin, add `-Des.cluster.name=my-cluster` to VM options when it's just `cluster.name` in `elasticsearch.yml` file.

You can also add `run` task to your Gradle build script and run you app using `gradle build run` command.

```groovy
task run(type: JavaExec, dependsOn: classes) {
    main = 'org.elasticsearch.bootstrap.ElasticsearchF'
    classpath sourceSets.main.runtimeClasspath
    classpath configurations.runtime
}
```

## Define modules as alternative solution

Usually plugin is something more than just REST endpoint and it's better to split our plugin on modules. First module can be our REST hello endpoint. The Hello module class must be inherited from `org.elasticsearch.common.inject.AbstractModule` and have overridden `configure` method to register handler

```scala
def override configure():Unit = bind(classOf[HelloRestHandler]).asEagerSingleton
```

You can register your modules by overriding `Collection<Class<? extends Module>> modules()` java method of the plugin class.
First of all we need to define list of modules our plugin contains. We have to convert Scala list to Java list to satisfy Java interface. When you import `scala.collection.JavaConverters` conversion will happen implicitly, but according to [Effective Scala][effective-scala] book by Twitter it's recommended to use explicit `asJava` method, aiding reader. Finally the method will look like:

```scala
def override modules() {
  List(
       classOf[HelloModule],
       classOf[UselessModule]
  ).asJava
}
```

## Release!

Next step will be release our plugin to be able to install plugin using standard elasticsearch commnand. This command will look like `./bin/plugin --install mylifeecho/hello-plugin/0.0.1`. Version number of course should be according [Semantic Versioning][semver], but keep in mind that your plugin builded for elasticsearch 1.3.x may not work on elasticsearch 1.4.x. In my case I had issue due to changes in interface of `BaseRestHandler` contructor between 1.3 and 1.4 versions. So you probably would like to have plugin version per minor elasticsearch version like [these guys][aws-plugin] do.
When you run `./bin/plugin --install` command elasticsearch will try to access `download.elastic.co` first and than maven central in order to download your plugin.
Follow [OSSRH Guide][ossrh] to deploy plugin and take a look at [OSSRH Gradle][ossrh-gradle]. After that you can install your plugin.

Restart elasticsearch after installation. Output will be similar to

```
[2015-05-31 21:34:56,276][INFO ][node    ] [Varys] version[1.5.2], pid[8572], build[62ff986/2015-04-27T09:21:06Z]
[2015-05-31 21:34:56,277][INFO ][node    ] [Varys] initializing ...
[2015-05-31 21:34:56,296][INFO ][plugins ] [Varys] loaded [hello-plugin], sites []
[2015-05-31 21:34:59,593][INFO ][node    ] [Varys] initialized
[2015-05-31 21:34:59,740][INFO ][node    ] [Varys] starting ...
```

You can find the source code of hello plugin on [Guthub][hello-src] or install plugin with the command `bin\plugin --install hello-plugin --url http://bit.ly/1ADC0bB`.

In my next blog post we are going to add support of new script language into elasticsearch.

[4sq]: https://github.com/foursquare/es-scorer-plugin
[idea]: https://www.jetbrains.com/idea/
[why-gradle]: https://google.com/?q=why+gradle
[mvn2gradle]: http://maruhgar.blogspot.nl/2010/12/converting-maven-project-to-gradle.html
[scala]: http://www.scala-lang.org/download/
[gradle]: https://gradle.org/
[es]: https://www.elastic.co/downloads/elasticsearch
[bigdesk]: https://github.com/lukas-vlcek/bigdesk
[guice]: https://github.com/google/guice
[effective-scala]: http://twitter.github.io/effectivescala/
[semver]: http://semver.org/
[aws-plugin]: https://github.com/elastic/elasticsearch-cloud-aws
[ossrh]: http://central.sonatype.org/pages/ossrh-guide.html
[ossrh-gradle]: http://central.sonatype.org/pages/gradle.html
[hello-src]: https://github.com/mylifeecho/hello-plugin
