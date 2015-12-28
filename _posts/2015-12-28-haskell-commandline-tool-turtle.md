---
layout: post
title: "Command-line tool in Haskell step-by-step"
description: "It's easy to write pretty nice command-line tool in Haskell with small amount of code."
category: dev
tags: [ Haskell, Turtle ]
---
{% include JB/setup %}

After finishing [Introduction to Functional Programming][fp-edx] course on *edx* (which was mainly focused on Haskell) I was very excited about this language. Concise, powerful type inference system, designed to make functional programming easy to write and read. I have even found out that it has a few concepts that corresponds with quantum physics. And actually I'm not the only one. There is even library called [quipper][quipper] to program quantum computer (which does not exists yet) based on Haskell... but it's a topic for another article. Then I started to feel that Haskell is awesome but it's most likely too complex and too academic to be applied for regular tasks. You remember "There is no bad or good languages, tools, technologies, etc. You should choose appropriate technology to solve your problem in most efficient way." That feeling is described perfectly in [this][hhh] blog post. I started to look for proofs and opinions on the Internet to confirm my bad feeling, but instead I found out that Haskell is mature enough to become technology of chose for many applications. And this time we are going to build simple command-line tool on Haskell and see how easy it is.

## Step-by-step tutorial:
### Step 1. Create `any-tool` using Stack

I'm going to use [stack][stack] package manager to create, build and run project.
Run the following commands to create project:

```
$ stack new any-tool simple
$ cd any-tool
```

`any-tool` is name of the project and `simple` is template name. You can check list of templates using `stack templates` command and choose most appropriate. For example it can be `new-template`. It will create project with app, library and tests folders with simplest code you can have.

### Step 2. Add Turtle library for command-line tools

 [Turtle][turtle] is very nice library to write command-line tools in Haskell. Open `any-tool.cabal` file and add change yaml to have the following:

```yaml
build-depends: base >= 4.7 && < 5
             , turtle
```
Make sure you have the following header on top of `Main.hs`

```haskell
{-# LANGUAGE OverloadedStrings #-}
module Main where
import Turtle
```

### Step 3. Write first command

We are going to define default behavior when user types `any-tool` command in terminal

```haskell
mainSubroutine :: IO ()
mainSubroutine = echo "Any tool just works!"
```

Then create parser for your subroutine using `pure` function without any argument or flag-based options. It will run `mainSubroutine` by default

```haskell
parseMain :: Parser (IO ())
parseMain = pure mainSubroutine

parser :: Parser (IO ())
parser = parseMain

main :: IO ()
main = do
    cmd <- options "Just any tool you could imagine" parser
    cmd
```

Let's build it, run it and see output.
If you run your tool using `stack` as I do you can pass arguments after `--`. Thus `stack exec any-tool -- subcommand --help` is equivalent of `any-tool subcommand --help`.

```
$ stack build
$ stack exec any-tool
Any tool just works!

$ stack exec any-tool -- --help
Just any tool you could imagine

Usage: any-tool

Available options:
  -h,--help                Show this help text
```

Output is expected, help text pretty descriptive. Looks OK but not very impressive so far. Let's extend command-line tool to have some extra functionality.

### Step 5. Print version

To print version number of your tool you should import `Data.Version` module and `version` function from the `Paths_{program_name}` file created by Cabal with metadata and module to retrieve version inside.

```haskell
import Paths_any_tool (version)
import Data.Version (showVersion)
```

Then you can print version

```haskell
version' :: IO()
version' = putStrLn (showVersion version)
```
### Step 6. Add subcommand to print version information

Let's add a new subcommand to print verbose version information `any-tool version`

```haskell
verboseVersion :: IO()
verboseVersion = do
                 version'
                 echo "Verbose version information"
```

and wrap it using Turtle API to print help for new command. Version subcommand does not have any arguments or options so we will use `pure` function again.

```haskell
parseVersion :: Parser (IO ())
parseVersion =
    (subcommand "version" "Show the Awesome tool version" (pure verboseVersion))
```

Last step is to add it to main `parser`:

```haskell
parser = parseMain <|> parseVersion
```

Build and run the tool:

```
$ any-tool version
0.1.0.0
Verbose version information

$ any-tool version --help
Show any-tool version

Usage: any-tool version

Available options:
  -h,--help                Show this help text
```

### Step 7. Add subcommand with arguments and options

This step-by-step guide would be not complete without subcommand with command line arguments and flag-based options. Let's build subcommand that prints input text several times. By default it will print text once but we will add flag-based option to specify number of times input text should be printed. For instance: `any-tool print --times 5 "text to print"`.

`Maybe` monad and pattern matching make implementation of options trivial. Function will accept tuple of `Maybe Int` as first element with number of times we have to repeat `Text` passed as second element of the tuple.

```haskell
printText :: (Maybe Int, Text) -> IO()
printText (Nothing, text) = echo text
printText ((Just i), text) = replicateM_ i (echo text)
```

Now we need to define wrapper for `printText` function. We can take a look at [Turtle documentation][turtle-options] for more information.
To create optional `--times` or `-n` option we will use `optional` to convert result of `optInt` to Maybe monad.
To create positional argument we will use `argText`.

```haskell
printArgs :: Parser (Maybe Int, Text)
printArgs = (,) <$> optional (optInt "times" 'n' "Number of times")
                <*> (argText "text" "Text to print")
```

Then combine `printText` and `printArgs` with subcommand name and help message

```haskell
parsePrint :: Parser (IO ())
parsePrint = fmap printText
    (subcommand "print" "Print specified text specified number of times" printArgs)
```

and last step is to extend main parser

```haskell
parser = parseMain <|> parseVersion <|> parsePrint
```

Here we are. Build and run.

```
$ any-tool print
Usage: any-tool print [-n|--times TIMES] TEXT
```

We forgot required argument and turtle shows us how to use `print` subcommand.

```
$ any-tool print "text to print"
text to print

$ any-tool print --times 3 "it will appear 3 times"
it will appear 3 times
it will appear 3 times
it will appear 3 times
```

And last but not least take a look at result help for main command and for `print` subcommand

```
$ any-tool -h
Just any tool you could imagine

Usage: any-tool ([version] | [print])

Available options:
  -h,--help                Show this help text

Available commands:
  version
  print
```

```
$ any-tool print --help
Print specified text specified number of times

Usage: any-tool print [-n|--times TIMES] TEXT

Available options:
  -h,--help                Show this help text
  -n,--times TIMES         Number of times
  TEXT                     Text to print
```

### Step 8. Extend your tool following Turtle tutorial

Strictly speaking it's better not to mix parsing logic and logic of your application. One of the improvements is to define command data type so parser will return command and then main will execute command. Something like presented below:

```haskell
data Command = Add Int | Subtract Int | Divide Int | Multiply Int

-- and then have something like this
main = do
    cmd <- options "Just any tool you could imagine" commandParser
    exec cmd
```

## Conclusions

At the matter of fact Turtle is mainly focused on writing shell scripts on Haskell but it also allows us to build pretty advanced command-line tools. If you are interested in more information about Turtle take a look at [official site] and [tutorial][turtle-tutorial].

You can find source code from this blog post [here on github][src]

Happy coding!

[fp-edx]: https://www.edx.org/course/introduction-functional-programming-delftx-fp101x-0
[quipper]: http://www.mathstat.dal.ca/~selinger/quipper/
[turtle]: https://hackage.haskell.org/package/turtle
[turtle-tutorial]: https://hackage.haskell.org/package/turtle-1.2.4/docs/Turtle-Tutorial.html
[turtle-options]: https://hackage.haskell.org/package/turtle-1.2.4/docs/Turtle-Options.html#g:2
[src]: https://github.com/mylifeecho/any-tool
