# vinyasa

[Give your clojure workflow more flow](http://z.caudate.me/give-your-clojure-workflow-more-flow/)

[![Build Status](https://travis-ci.org/zcaudate/vinyasa.svg?branch=master)](https://travis-ci.org/zcaudate/vinyasa)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents** [[doctoc](https://github.com/thlorenz/doctoc)]

- [vinyasa](#vinyasa)
	- [Whats New](#whats-new)
	- [Installation](#installation)
	- [Quickstart:](#quickstart)
		- [pull](#pull)
		- [lein](#lein)
		- [reimport](#reimport)
		- [inject](#inject)
	- [License](#license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Whats New

#### 0.2.2
- breaking changes to `vinyasa.inject/inject`, see [example](#inject)
- a new helper macro `vinyasa.inject/in` for prettier imports, see [example](#installation)

#### 0.2.0

vinyasa has now been [repackaged](https://github.com/zcaudate/lein-repack). Functionality can now be accessed via seperate dependencies:

```clojure
[im.chit/vinyasa.inject "VERSION"]
[im.chit/vinyasa.pull "VERSION"]
[im.chit/vinyasa.lein "VERSION"]
[im.chit/vinyasa.reimport "VERSION"]
```

Or all of them together:

```clojure
[im.chit/vinyasa "VERSION"]
```

#### 0.1.9
Changed `vinyasa.lein` according to [comments](http://z.caudate.me/clojure-dynamic-languages-creativity-and-simplicity/) on blog.

WARNING: There are [issues](https://github.com/zcaudate/vinyasa/issues/3) with adding leiningen as a dependency. It should be disabled if it causes problems.

#### 0.1.8
Breaking changes to `reimport`. Now reimport is used like this:

```clojure
(reimport :all)  ;; compile import all symbols into namespace

(reimport 'com.example.Util
          '[net.example Hello World]
          false) ;; do not import symbols
```
## Installation

Add `vinyasa` to your `profiles.clj` (located in `~/.lein/profiles.clj`) as well as your version of leiningen. Please note the issue with `vinyasa.lein` with light table as well as other libraries. You may need to disable `vinyasa.lein` and `leiningen` if there are problems.

`inject` has been quite popular due this [article](http://dev.solita.fi/2014/03/18/pimp-my-repl.html). It's main use is to create extra symbols in a particular namespace, namely `clojure.core`. Customisation of your namespace can be done by injecting functions in seperate namespaces into either `clojure.core` or another namespace. This is typically done through the `:injections` value in your `~/.lein/profiles.clj` file.

Previously, injecting functions into `clojure.core` with a prefix was the way to avoid clashing names. This is still a good option but it may not be the best option going forward. One [issue](https://github.com/zcaudate/vinyasa/issues/9) noted that `(in-ns <namespace>)` call will not automatically call `(refer-clojure)` and so will not import methods from `clojure.core`. 

As such `inject` has undergone quite a bit of refinement since version `0.2.2`. 
 
- Firstly, it is suggested that a short namespace be used instead of adding a prefix. For example, instead of typing `>pprint`, we type `>/pprint` or by default, `./pprint` (the suggested default namespace is now "`.`").
    
- Secondly, a macro should be defined such that the user can have less quoting as well as have familiar ways that are provided by the `ns` macro. `vinyasa.inject/in` enables declarations in a more dsl-like nature. 
  
Here is an example of a typical `profiles.clj` configuration:

```clojure
{:user {:plugins [...]        
         :dependencies [[spyscope "0.1.4"]
                        [org.clojure/tools.namespace "0.2.4"]
                        [leiningen #=(leiningen.core.main/leiningen-version)]
                        [im.chit/iroh "0.1.11"]
                        [io.aviso/pretty "0.1.8"]
                        [im.chit/vinyasa "0.2.2"]]
         :injections [(require 'spyscope.core)
                      (require '[vinyasa.inject :as inject])
                      (require 'io.aviso.repl)
                      (inject/in ;; the default injected namespace is `.` 

                                 ;; note that `:refer, :all and :exclude can be used
                                 [vinyasa.inject :refer [inject [in inject-in]]]  
                                 [vinyasa.lein :exclude [*project*]]  

                                 ;; imports all functions in vinyasa.pull
                                 [vinyasa.pull :all]      

                                 ;; same as [cemerick.pomegranate :refer [add-classpath get-classpath resources]]
                                 [cemerick.pomegranate add-classpath get-classpath resources] 
                                 
                                 ;; inject into clojure.core 
                                 clojure.core
                                 [iroh.core .> .? .* .% .%>]
                                 
                                 ;; inject into clojure.core with prefix
                                 clojure.core >
                                 [clojure.pprint pprint]
                                 [clojure.java.shell sh])}}
```

The following vars will now be created under the `.` namespace:

```clojure
user=> ./
./>ns             ./>var            ./add-classpath   ./apropos         ./dir             ./doc
./find-doc        ./get-classpath   ./inject          ./inject-in       ./pprint          ./pst
./pull            ./refresh         ./resources       ./root-cause      ./sh              ./source
```

Reflection macros: `.>` `.?` `.*` `.%` `.%>` will be created in `clojure.core`

```clojure
user=> (.? "" :name)
("CASE_INSENSITIVE_ORDER" "HASHING_SEED" "charAt" "checkBounds" "codePointAt" "codePointBefore" "codePointCount" "compareTo" "compareToIgnoreCase" "concat" "contains" "contentEquals" "copyValueOf" "endsWith" "equals" "equalsIgnoreCase" "format" "getBytes" "getChars" "hash" "hash32" "hashCode" "indexOf" "indexOfSupplementary" "intern" "isEmpty" "lastIndexOf" "lastIndexOfSupplementary" "length" "matches" "new" "offsetByCodePoints" "regionMatches" "replace" "replaceAll" "replaceFirst" "serialPersistentFields" "serialVersionUID" "split" "startsWith" "subSequence" "substring" "toCharArray" "toLowerCase" "toString" "toUpperCase" "trim" "value" "valueOf")
```

Prefixed vars `>pprint` and `>sh` will be created in `clojure.core`

```clojure
user=> >
>    >=   >pprint   >sh
```

*NOTE* Its very important that `leiningen` is in your dependencies for `vinyasa.lein` and `vinyasa.import` as `lein` and `reimport` have dependencies on leiningen functions

## Quickstart:

Once `profiles.clj` is installed, run `lein repl`.

```clojure
> (./lein)    ;; => entry point to leiningen
> (./reimport) ;; => dynamically reloads *.java files
> (./pull 'hiccup) ;; => pull repositories from clojars
> (./inject 'clojure.core '[[hiccup.core html]]) ;; => injects new methods into clojure.core
> (html [:p "Hello World"]) ;; => injected method
;;=> "<p>hello world</p>"
```

### pull

How many times have you forgotten a library dependency for `project.clj` and then had to restart your nrepl? `pull` is a convienient wrapper around the `pomegranate` library:

```clojure
> (require 'hiccup.core)
;; => java.io.FileNotFoundException: Could not locate hiccup/core__init.class or hiccup/core.clj on classpath:

> (require 'hiccup.core)
> (pull 'hiccup)
;; => {[org.clojure/clojure "1.2.1"] nil,
;;     [hiccup "1.0.4"] #{[org.clojure/clojure "1.2.1"]}}

> (use 'hiccup.core)
> (html [:p "hello World"])
;; => "<p>hello World</p>"

> (pull 'hiccup "1.0.1")
;; => {[org.clojure/clojure "1.2.1"] nil,
;;     [hiccup "1.0.1"] #{[org.clojure/clojure "1.2.1"]}}
```
### lein

Don't you wish that you had the power of leiningen within the repl itself? `lein` is that entry point. You don't have to open up another terminal window anymore, You can now run your commands in the repl!

```clojure
> (lein)
;; Leiningen is a tool for working with Clojure projects.
;;
;; Several tasks are available:
;; check               Check syntax and warn on reflection.
;; classpath           Write the classpath of the current project to output-file.
;; clean               Remove all files from paths in project's clean-targets.
;; cljsbuild           Compile ClojureScript source into a JavaScript file.
;;
;;  .....
;;  .....

> (lein install)     ;; Install to local maven repo

> (lein uberjar)     ;; Create a jar-file

> (lein push)        ;; Deploy on clojars (I am using lein-clojars plugin)

> (lein javac)       ;; Compile java classes (use vinyasa.reimport instead)

```

### reimport

Don't you wish that you could make some changes to your java files and have them instantly loaded into your repl without restarting? Well now you can!

For example, in project.clj, you have specified your `:java-source-paths`

```clojure
(defproject .....
   :source-paths ["src/clojure"]
   :java-source-paths ["src/java"]
   :java-test-paths ["test/java"]    ;; *.java files that are not included in package
   ....)
```

and you have a file `src/java/testing/Dog.java`

```java
package testing;
public class Dog{
  public int legs = 3;
  public Dog(){};
}
```

You can load it into your library dynamically using `reimport`

```clojure
(reimport 'testing.Dog)
;;=> 'testing.Dog' imported from <project>/target/reload/testing/Dog.class

(.legs (Dog.))
;; => 3
```

You can then change legs in `testing.Dog` from `3` to `4`, save and go back to your repl:

```clojure
(reimport '[testing Dog]) ;; supports multiple classes
;;=> 'testing.Dog' imported from <project>/target/reload/testing/Dog.class

(.legs (Dog.))
;; => 4
```

If you have more files, ie. copy your Dog.java file to Cat.java and do a global replace:

```clojure
(reimport) ;; will load all classes into your namespace
;;=> 'testing.Dog' imported from <project>/target/reload/testing/Dog.class
;;   'testing.Cat' imported from <project>/target/reload/testing/Cat.class

(.legs (Cat.))
;; => 4
```

Now the pain associated with mixed clojure/java development is gone!

### inject

I find that when I am debugging, there are additional functionality that is needed which is not included in clojure.core. The most commonly used function is `pprint` and it is much better if the function came with me when I was debugging.

The best place to put all of these functions in in the `clojure.core` namespace
`inject` is used to add additional functionality to namespaces so that the functions are there right when I need them. Inject also works with macros and functions (unlike `intern` which only works with functions):

```clojure
> (inject '[clojure.core [clojure.repl dir]])
;; => will intern #'clojure.repl/dir to #'clojure.core/dir

> (clojure.core/dir clojure.core)
;; *
;; *'
;; *1
;; *2
;; *3
;; *agent*
;; *allow-unresolved-vars*
;; *assert*
;;
;; ...
;; ...
```

`inject` can also work with multiple entries:

```clojure
> (inject '[clojure.core [clojure.repl doc source]])
;; => will create the var #'clojure.core/doc and #'clojure.core/source
```

`inject` can also take a prefix:

```clojure
> (inject '[clojure.core >> [clojure.repl doc source]])
;; => will create the var #'clojure.core/>>doc and #'clojure.core/>>source
```

`inject` can use vector bindings to directly specify the name

```clojure
> (inject '[clojure.core >> [clojure.repl doc [source source]])
;; => will create the var #'clojure.core/>>doc and #'clojure.core/source
```

## License

Copyright © 2014 Chris Zheng

Distributed under the MIT License
