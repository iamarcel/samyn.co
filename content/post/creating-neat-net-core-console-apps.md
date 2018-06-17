+++
title = "Creating Neat .NET Core Command Line Apps"
date = 2016-09-06T16:15:17
math = false
highlight = true
tags = ["development"]
+++

Every reason to get more HackerPoints™ is a good one, so today we're going to
write a neat command line app in .NET Core! The Common library has a really cool
package `Microsoft.Extensions.CommandlineUtils` to help us parse command line
arguments and structure our app, but sadly it's undocumented.

No more! In this guide, we'll explore the package and write a really neat
console app. We'll get good practices, a help system and argument parsing for
free. Oh, it also involves ninjas. Insta-win.

The code is [all up on GitHub](https://github.com/iamarcel/dotnet-neat-console-app) so you can look at it there, clone the project and
play around for yourself. I've also created a little framework you can use for
your own projects, as a starting point for creating neatly structured neat
console apps.

Ready? Cool, me too.

<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#orgheadline4">1. Introduction to the package</a>
<ul>
<li><a href="#orgheadline1">1.1. What can we do?</a></li>
<li><a href="#orgheadline2">1.2. Structure of your application</a></li>
<li><a href="#orgheadline3">1.3. Getting help</a></li>
</ul>
</li>
<li><a href="#orgheadline8">2. Creating/Becoming a Console Ninja</a>
<ul>
<li><a href="#orgheadline5">2.1. The Application</a></li>
<li><a href="#orgheadline6">2.2. A Command</a></li>
<li><a href="#orgheadline7">2.3. Arguments and Options</a></li>
</ul>
</li>
<li><a href="#orgheadline9">3. That's it! (for now)</a></li>
<li><a href="#orgheadline10">4. Further Reading</a></li>
</ul>
</div>
</div>

# Introduction to the package<a id="orgheadline4"></a>

Basically, the `CommandLineUtils` package parses your arguments and helps you structure the different parts of your application. It makes it very easy to allow your app to accept all kinds of parameters, according to the conventions you probably know and love. 

## What can we do?<a id="orgheadline1"></a>

Let's assume we're creating an app called `ninja`. I'll list a couple of possible commands while I introduce you to the terminology used in the package. Whenever a word like Command, Option or Argument is capitalized, that means I'm talking about its specific definition regarding to the `CommandLineUtils` package.

-   `ninja hide` (simple Command)
-   `ninja weapon list` (nested Command)
-   `ninja attack --exclude civilians` (simple Command with Option)
-   `ninja weapon add {weaponType}` (nested Command with Argument)

Of course, we can start mixing and matching to make really powerful types of
commands. But once you've got the basics, everything else is easy-peasy.

## Structure of your application<a id="orgheadline2"></a>

So if we look at what we have above, the structure of our application will look
like this:

-   Root Command (`ninja`)
    -   Argument(s) (positional parameters, like `weaponType`)
    -   Option(s) (parameters that have a name, specified like `--exclude
            civilians`)
    -   More commands like `hide` and `weapon`, with their own Arguments and Options

## Getting help<a id="orgheadline3"></a>

Help is essential to any application, especially if it's a command line
application. The package is super-helpful here: you just specify some
explanation with yout Commands, Arguments and Options, and it'll spit out neatly
formatted and organized help when your user needs it.

# Creating/Becoming a Console Ninja<a id="orgheadline8"></a>

Let's dive in and start doing some coding.

## The Application<a id="orgheadline5"></a>

We create our application by creating a `CommandLineApplication`.

```cs
var app = new CommandLineApplication();
app.Name = "ninja";
app.HelpOption("-?|-h|--help");
```

The second line is the name of our application. It'll be used in the help
output.

The third line specifies which options trigger the help output. The syntax of
the string is self-explanatory: use either "-?", "-h" or "&#x2013;help" as a parameter
for the app to open up the help.

To make it actually do something, we define a function in
`onExecute(Func<int>)`. Note that it expects an `int` as return value (the error
code). To run the actual `CommandLineApplication`, call `app.Execute(string[]
args)`. You can just pass along the arguments you got from the `Main` entry
point of your console app.

```cs
app.OnExecute(() => {
        Console.WriteLine("Hello World!");
        return 0;
    });

app.Execute(args);
```

Now, "Hello World!" will be printed when you run the app. If you specify, for
example `--help`, an (empty) help thing will show up. Neat uh?

## A Command<a id="orgheadline6"></a>

This is how we create a Command:

```cs
app.Command("hide", (command) =>
    {
        command.Description = "Instruct the ninja to hide in a specific location.";
        command.HelpOption("-?|-h|--help");

        var locationArgument = command.Argument("[location]",
                                   "Where the ninja should hide.");

        command.OnExecute(() =>
            {
                var location = locationArgument.HasValue()
              ? locationArgument.Value()
              : "under a turtle";
                Console.WriteLine("Ninja is hidden " + location);
            });
    });
```

`app.Command` takes a command name and a configuration function as arguments.
Note that the type of `command` in the configuration function is
`CommandLineApplication`. Yup, that's how easy it is to start nesting commands.

The `Description` will be used in the help output and the call to `HelpOption`
is required in order to enable those arguments.

Let's see what happens now:

```sh
$ ninja hide
Ninja is hidden under a turtle

$ ninja hide "on top of a street lamp"
Ninja is hidden on top of a street lamp

$ ninja -?
Usage: ninja [options] [command]

Options:
  -?|-h|--help  Show help information

Commands:
  hide  Instruct the ninja to hide in a specific location.

Use "ninja [command] --help" for more information about a command.

$ ninja hide --help
Usage: ninja hide [arguments] [options]

Arguments:
  [location]  Where the ninja should hide.

Options:
  -?|-h|--help  Show help information
```

Neat!

## Arguments and Options<a id="orgheadline7"></a>

You saw that with the `command.Argument` function we, well, created an argument!
Just pass on the name and description (again, for the help output) and
`CommandLineUtils` will do its magic. Make sure you store that variable
somewhere; when executing you'll need it to get the value of your arguments.

Our ninja needs more powers. Powers with Options.

```cs
app.Command("attack", (command) =>
    {
        command.Description = "Instruct the ninja to go and attack!";
        command.HelpOption("-?|-h|--help");

        var excludeOption = command.Option("-e|--exclude <exclusions>",
                                "Things to exclude while attacking.",
                                CommandOptionType.MultipleValue);

        var screamOption = command.Option("-s|--scream",
                               "Scream while attacking",
                               CommandOptionType.NoValue);

        command.OnExecute(() =>
            {
                var exclusions = excludeOption.Values;
                var attacking = (new List<string>
                {
                    "dragons",
                    "badguys",
                    "civilians",
                    "animals"
                }).Where(x => !exclusions.Contains(x));

                Console.Write("Ninja is attacking " + string.Join(", ", attacking));

                if (screamOption.HasValue())
                {
                    Console.Write(" while screaming");
                }

                Console.WriteLine();

                return 0;

            });
    });
```

Recognize the syntax? The first argument in `command.Option` defines the
template, the second is a description and the third defines the type of
argument. This can be `SingleValue`, `MultipleValue` or `NoValue` (that's for
toggles, use `myOption.HasValue()` to check if it has been set).

The user can specify a value for options using either a space or an equals sign
between the option name and its value: `--exclude civilians -e=animals` works
perfectly (yes, also in a single command since the option type is
`MultipleValue`).

Note that an Option is Option-al so be sure handle the case where it's not given.
Also, only the first word is parsed as part of the option. If you type `-e
civilians animals`, the parser wil throw an exception.

The following commands will all work as you'd expect:

```sh
$ ninja attack -?
$ ninja attack --scream
$ ninja attack -e dragons -s --exclude=animals
```

Neat!

# That's it! (for now)<a id="orgheadline9"></a>

Well done! You got the basics down and are now ready to create really cool
console applications. All hail the command line! Now go out and impress your
friends because you've accumulated quite some HackerPoints™ now. (just don't
forget change your color scheme so there's green text on a black background)

But soon you'll start creating really big, useful impressive console apps and it
could get a bit messy to keep this all in your `Main` function. So what's the
best way to neatly *organize* our neat app? I got ya covered. The guys at Entity
Framework did a really good job in their tooling; I analyzed their sources with
my lazer eyes and wrote you a simple guide (and a little framework!).

[Read on about structuring neat console apps neatly](https://gist.github.com/iamarcel/9bdc3f40d95c13f80d259b7eb2bbcabb).

Shoutout to [4D Vision](http://4dvision.be) for letting me write this while working as a student for
them! They build custom software for complex processes of companies and
organizations; not to mention they're really cool people. This article is also
part of their internal documentation now.

Thanks for reading! If you wanna keep up with my work, [follow me on twitter](https://twitter.com/mrclsmn)
because that's where the cool guys are at, right?

# Further Reading<a id="orgheadline10"></a>

-   [Essential .NET - Command-Line Processing with .NET Core 1.0](https://msdn.microsoft.com/en-us/magazine/mt763239.aspx)
-   [Microsoft.Extensions.CommandLineUtils API Docs](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/Extensions/CommandLineUtils)
-   [The Entity Framework Core command line utility (source code)](https://github.com/aspnet/EntityFramework/tree/dev/src/Tools.Console)
