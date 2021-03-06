js-class-loader
===============

(Java) A Free Software tool for bundling and serving large Javascript codebases with built-in dependency detection. 
Fast, tested, easy to integrate and does not use the words rockstar or ninja, or have a website filled with big gradient 
icons of robot kittens playing banjos. (but then it's not a javascript layout framework so I suppose it wouldn't anyway.)

Rules of JS-Class-Loader
------------------------

1. Class names must match file names
2. Folder structure must match package structure
3. File names must not have extra dots in them (dashes and underscores are ok though)
4. You must have a well defined way of extending classes

Having said that, you can still include 3rd party libraries and modules in your bundle that don't conform to this scheme. 
You just need to explicitly include them in your code somewhere, you can't just rely on dependency detection to work out
that you need them.

What is it?
-----------

JS-Class-Loader is a java tool for managing large javascript codebases. It is not a generic tool like WRO4J or RequireJS
that can be used on any codebase given a few modifications. This tool requires your source files to be organised in the
Java style, where folders match package names and filenames match class names. If your code is organised this way then
JS-Class-Loader will detect dependencies without you having to declare them at all, bundle your code or generate script
tags for development, and also provide diagnostics like dependency graphs, lists of unused files and validation that your files
contain the classes you think they do. (it's all too easy to copy-and-paste a function onto the wrong prototype in JS)

JS-Class-Loader works as either a command line tool, a Maven Mojo, a Gradle task or a Servlet. The command line or java runners
can be integrated into any other build system or just used ad-hoc. 

A recommended approach would be to use the servlet for dev and the generated static bundle file for test and production installations.

It can be used to manage your own code as well as 3rd party modules. Third party modules may have to be listed as an explicit
import() but they will be added to the bundle in the same way.

See the wiki at http://github.com/damonsmith/js-class-loader for all the info and examples.

Getting started
---------------

The simplest case on the command line:
java -jar js-class-loader.jar --seed-file Main.js --output-file bundle.js

This will use Main.js as the seed file, the current folder as the only source folder and generate a bundle of everything
that is required for Main.js.

The way that it works is that it first finds all js files in the current source tree. It then matches package and class 
usages in source files to find runtime dependencies. It also parses any use of the extend function to track parse-time
dependencies and make sure they are loaded into the bundle before the subclass that requires them.

See the docs for more info on running JS-Class-Loader and integrating it into your app.

Dependency Detection
--------------------

The JS-Class-Loader in it's natural state will just detect all of your dependencies starting with a seed file, (either code
or config) generate bundles at request-time for dev and generate static bundles for production.

The way it does this is:
1. It looks at the names and paths of all of the files in your configured source locations.
2. Starting with the seeds, it parses any file it finds for a string that looks like a known class type, so if your source path is

/src

and you have a file in /src/com/mycompany/tools/Loader.js

then if your seed or any other included file uses the string: com.mycompany.tools.Loader then Loader.js will be included in the
bundle.

3. Parse-time dependencies
OO Javascript usually extends sub-classes at parse time, like this:

```javascript
com.mycompany.tools.Loader = function() { /*constructor */ };
 
extend(com.mycompany.tools.Loader, com.mycompany.app.BaseLoader);
```

So when JS-Class-Loader sees a line like this, it includes src/com/mycompany/app/BaseLoader.js and makes sure it 
appears before Loader.js in the bundle file so that the parse time dependency works. 

You can provide a config file and customise the regex that matches parse-time dependencies too, or provide more and different ones
to customise it to your own syntax.


Comparison with other tools
---------------------------

RequireJS is a popular tool for doing this sort of thing. Here are some reasons why this project still exists and isn't
considered superceded by requireJS:

RequireJS needs nodejs or rhino to run and takes a loooong time to generate bundles. Even running from nodejs the docs
talk about taking 12 seconds to generate a bundle. JS-Class-Loader is so fast that you can hit save on a js file, hit reload on your
browser and a few milliseconds later the bundle will be regenerated and served to the page.

Of course with RequireJS you can lazy load your files in dev mode and have it traverse your dependency tree that way.
For large codebases that becomes less and less practical, meaning you may find that developers are excited at first at how everything
just works via magic and then more and more frustrated as the project goes on and the dev cycle gets slower and slower.
This tool provides a solution where - if you can organise your files in a standard Java-like way - then this tool can 
do all the dependency management, dev tooling and development bundling for you very quickly and scale to any size.

WRO4J also bundles javascript and other web assets and is generally very useful. It does not however provide anything
to manage dependencies. It is fine if you have no parse-time dependencies and you just want to bundle everything, but 
if that is not the case then you will end up doing a lot of manual handling of your javascript.


History of the JS-Class-Loader
------------------------------

JS-Class-Loader was written as a complete, free and open implementation of tools that exist in-house in many enterprise java development
teams that I have worked in. The tooling is designed to scale to enterprise levels for codebases that have megabytes of js code, 
tens of thousands of files, loads of source locations, and is flexible enough that most config files for layouts or app structure
can be used as seed files.

The project started off as a fork of the Caplin Trader javascript bundler but has been rewritten from scratch.

