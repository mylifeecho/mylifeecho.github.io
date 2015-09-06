---
layout: post
title: "Brainfuck scripting support for elasticsearch"
description: "" 
category: dev
tags: [ Scala, elasticsearch, Brainfuck ]
---
{% include JB/setup %}

We as a programmers like some languages and some of them... are not interested in at all. It hurts so badly when you don't have an option to use your lovely language when you clearly see how beautiful solution of your problem will be. 

Let's say you work with elasticsearch and you need to write some script. You take a look at the [list of supported languages][es-list-supported-lang] out of the box, than at list of plugins which add other languages. But no, groovy, javascript, python or closure are in your blacklist. There is an option for you! Write your own plugin to extend elasticsearch functionality and you will be able to write scripts in prefered language. [Brainfuck][brainfuck] for instance!

This post is the continuation of the previous [Elasticsearch, Scala, Gradle. Writing plugin step-by-step][hw-plugin] post. We are going to add the support of another scripting language to elasticsearch. 

Brainfuck is "esoteric" language from 90s and its interpreter [looks][brainfuck-int] quite simple in Scala, just 40 lines (thanks to Peter Braun). 

Create plugin class as described in the [provious post][hw-plugin], replace `RestModule` by `ScriptModule` and implement `onModule` method body registering `addScriptEngine`

```scala
class BrainfuckPlugin extends AbstractPlugin {
  override def name(): String = "lang-brainfuck"

  override def description(): String = "Adds support for writing scripts in Brainfuck"

  def onModule(module: ScriptModule): Unit = module.addScriptEngine(classOf[BrainfuckScriptEngineService])
}
```

[hw-plugin]: http://mylifeecho.com/dev/elasticsearch-plugin-scala/
[brainfuck]: https://en.wikipedia.org/wiki/Brainfuck
[brainfuck-int]: http://peter-braun.org/2012/07/brainfuck-interpreter-in-40-lines-of-scala/
[es-list-supported-lang]: https://www.elastic.co/guide/en/elasticsearch/reference/1.7/modules-plugins.html#scripting
