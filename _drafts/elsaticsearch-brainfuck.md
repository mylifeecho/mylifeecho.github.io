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

This post is the continuation of the previous [Elasticsearch, Scala, Gradle. Writing plugin step-by-step][hw-plugin] post. We are going to add the support of another scripting language to elasticsearch. It is going to be simples and minimal working solution to evaluate brainfuck scripts. Don't you think to use it in production?!

Brainfuck is "esoteric" language from 90s and its interpreter [looks][brainfuck-int] quite simple in Scala, just 40 lines (thanks to Peter Braun). I'm going to use slightly changed version of his implementation.

Create plugin class as described in the [provious post][hw-plugin], replace `RestModule` by `ScriptModule`, change `name()` and `description()` to correspond the plugin purpose and implement `onModule` method body registering `addScriptEngine`. Final class will look like the following:

```scala
class BrainfuckPlugin extends AbstractPlugin {
  override def name(): String = "lang-brainfuck"
  override def description(): String = "Adds support for writing scripts in Brainfuck"
  def onModule(module: ScriptModule): Unit = module.addScriptEngine(classOf[BrainfuckScriptEngineService])
}
```

Create class `BrainfuckScriptEngineService` that extends `AbstractComponent` and implements `ScriptEngineService` interface. Implemnetation of `ScriptEngineService` is relatively straightforward: override `types` and `extensions` methods to return arrays of appropriate values to define supported scripting language. Final listing of the class without imports:

```scala
class BrainfuckScriptEngineService @Inject() (settings: Settings) extends AbstractComponent(settings) with ScriptEngineService {
  override def types(): Array[String] = Array("bf", "brainfuck")
  override def extensions(): Array[String] = Array("brainfuck")

  override def compile(script: String): AnyRef = script
  override def execute(compiledScript: scala.Any, vars: util.Map[String, AnyRef]): AnyRef = {
    BrainfuckEval.eval(compiledScript.asInstanceOf[String].toCharArray)
  }

  override def unwrap(value: AnyRef): AnyRef = value
  override def sandboxed(): Boolean = false
  
  override def executable(compiledScript: scala.Any, vars: util.Map[String, AnyRef]): ExecutableScript = {
    new BrainfuckExecutableSearchScript(compiledScript.asInstanceOf[String])
  }
  override def search(compiledScript: scala.Any, lookup: SearchLookup, vars: util.Map[String, AnyRef]): SearchScript = {
    new BrainfuckExecutableSearchScript(compiledScript.asInstanceOf[String])
  }
}
```

`compile(script:String)` method should return compiled version of the script but in our case we will return script itself since we implement support of brainfuck just using evaluator. 
`execute(compiledScript, vars)` has actual call to evaluate `compiledScript` but we will ignore `vars`.
`unwrap(value:AnyRef)` can return value as is, sandboxed() returns `false`, the rest of the method can be empty except 
two most interesting methods `executable` and `search` which have the same signature as `execute` method.

And they have to return instance of `ExecutableScript` and `SearchScript` respectively. For simplisity I will implement those interfaces in one class. For our minimal working solution we can implement `run` method which will evaluate script we passed to contractor and methods `runSomething` just do simple cast of the `run` results to proper type.

```scala
class BrainfuckExecutableSearchScript(script: String) extends ExecutableScript with SearchScript {
  override def unwrap(value: scala.Any): AnyRef = value
  
  override def run(): AnyRef = BrainfuckEval.eval(script.toCharArray)
  override def runAsFloat(): Float = run().asInstanceOf[Float]
  override def runAsLong(): Long = run().asInstanceOf[Long]
  override def runAsDouble(): Double = run().asInstanceOf[Double]
  
  /*...*/
}
```



[hw-plugin]: http://mylifeecho.com/dev/elasticsearch-plugin-scala/
[brainfuck]: https://en.wikipedia.org/wiki/Brainfuck
[brainfuck-int]: http://peter-braun.org/2012/07/brainfuck-interpreter-in-40-lines-of-scala/
[es-list-supported-lang]: https://www.elastic.co/guide/en/elasticsearch/reference/1.7/modules-plugins.html#scripting
