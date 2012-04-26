---
layout: post
title: Leiningen LessCSS plugin
---

As a side project to explore the [Clojure](http://www.clojure.org) programming language, I wrote a small plugin for compiling [LessCSS](http://lesscss.org) resources in a [Leiningen](https://github.com/technomancy/leiningen) project.

Leiningen is a very nice tool that helps you manage Clojure projects: you specify a project description file, and let Leiningen handle the classpaths, the download of needed dependencies and so on. A bit like what [Maven](http://maven.apache.org/) does in the Java world.

Leiningen is also extensible, allowing developers to use plugins for performing different tasks.

What I wrote is a plugin that compiles the LessCSS files present in a project so that they can be used by the application.

This is very useful if you want to integrate LessCSS in a web application developed with one of the many web frameworks available for Clojure like, for example, [Compojure](https://github.com/weavejester/compojure/wiki)

# Installation and usage

You just need to declare the `[lein-lesscss "1.0-SNAPSHOT"]` in the `:plugins` section of your `project.clj`. The plugin is available on [Clojars](http://clojars.org/) so it will automatically downloaded.

By running `lein` on your project root, you will see that a `lesscss` task is now available. 

You can then run `lein lesscss` to compile all the LessCSS `.less` files that are found in the `less` directory of your project tree.

The resulting CSS files will be then made available to the application which can reference them.

# Source code

Source code is available on [GitHub](https://github.com/fmancinelli/lein-lesscss). All contributions and feedbacks are welcome.

