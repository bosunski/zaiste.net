---
created_at: 2014-06-05
kind: article
publish: true
title: "Clojure CLI 101"
tags:
- clojure
- cli
---

The most convenient way of running a Clojure program is by creating a single,
standalone JAR file. It can be easily distributed and executed by end users
without the need of installing additional dependencies.

`lein` provides `uberjar` command that does most of work. There are, however, few
details that have to manually set up.

Let's start by creating an app with `lein new [name]`

```
lein new app
```

First, specify the main class namespace with `:main` and ensure it is compiled
ahead of time (AOT) with `:aot`.

```
(defproject app "0.1.0"
  ...
  :uberjar-name "app-standalone.jar"
  :aot  [app.core]
  :main app.core)¬
```

AOT can be also enabled only for the uberjar using `profiles` (thanks to Jan
Stępień for [the tip][8]).

```
(defproject app "0.1.0"
  ...
  :uberjar-name "app-standalone.jar"
  :profiles {:uberjar {:aot [app.core]}}
  :main app.core)
```

Optionally, you can specify the name for the jar that will be produced with
`:uberjar-name`. For the list of possible Leiningen parameters that can be used
in `projects.clj` check the following [sample.project.clj][1].

Next, tell Clojure to produce that main class from a (previously chosen)
namespace using `:gen-class` in its declaration along with `-main`
function defined inside.

```
(ns app.core
  (:gen-class))

(defn -main [& args]
  (println args))
```

Finally, run `lein uberjar` and then

```
java -jar target/app-standalone.jar param1 param2
```

It should produce: `(param1 param2)`.

`app-standalone.jar` provides interoperability, which means that this file can
be moved to other computer with Java installed and run on it the same way without
the need to install additional dependencies.

[tools.cli][2] provides tools to handle more complex scenerios while working with
command line arguments.

Let's start by adding it as a dependency to `project.clj`

```
(defproject app "0.1.0"
  :main app.core
  :aot [app.core]
  :dependencies [[org.clojure/clojure "1.6.0"]
                 [org.clojure/tools.cli "0.3.1"]])
```

Next, modify `core.clj`

```
(ns app.core
  (:require [clojure.tools.cli :refer [parse-opts]])
  (:gen-class))

(def cli-options
  ;; An option with a required argument
  [["-p" "--port PORT" "Port number"
    :default 80
    :parse-fn #(Integer/parseInt %)
    :validate [#(< 0 % 0x10000) "Must be a number between 0 and 65536"]]
   ;; A non-idempotent option
   ["-v" nil "Verbosity level"
    :id :verbosity
    :default 0
    :assoc-fn (fn [m k _] (update-in m [k] inc))]
   ;; A boolean option defaulting to nil
   ["-h" "--help"]])

(defn -main [& args]
  (parse-opts args cli-options))
```

Recompile with `lein uberjar` and run

```
java -jar target/app-standalone.jar param1 -h -v -p 40 param2 --invalid-opt
```

It should produce

```
{:options {:help true, :verbosity 1, :port 40}, :arguments [param1 param2], :summary   -p, --port PORT  80  Port number
  -v                   Verbosity level
  -h, --help, :errors [Unknown option: "--invalid-opt"]}
```


[1]: https://github.com/technomancy/leiningen/blob/stable/sample.project.clj
[2]: https://github.com/clojure/tools.cli
[8]: https://twitter.com/janstepien/status/482075401888743424
