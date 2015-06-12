---
layout: post
title: "Brainfuck script support for elasticsearch"
description: "" 
category: dev
tags: [ Scala, elasticsearch, Brainfuck ]
---
{% include JB/setup %}

This post is the continuation of the previous [Elasticsearch, Scala, Gradle. Writing plugin step-by-step][hw-plugin] post. We are going to add the support of another scripting language to elasticsearch. 
When you have quite simple script to write, it's usually not a big deal to learn a new language on the sufficient level to make this script. But in case you have to write something more complex (you have to be careful with performance impact of complex scripts even with built in languages) you may want to use language you are good in.

And the language we are going to add to elasticsearch is [brainfuck][brainfuck]! Brainfuck is "esoteric" language from 90s and its interpreter [looks][brainfuck-int] quite simple in Scala, just 40 lines (thanks to Peter Braun). 

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
