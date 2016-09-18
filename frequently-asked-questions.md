# Frequently Asked Questions

## General

### How do I get the current editor object?

`(pool/last-active)`

### How do I get every object of a given type?

`(object/instances-by-type ::type-of-your-object)`

### How do I delete every object of a given type?

`(map object/destroy! (object/instances-by-type ::type-of-your-object))`

### Within a command, how do I eval clojure code and call a function on its result?

1. Create a behavior with a trigger named `:editor.eval.clj.result.RESULT_TYPE`.Give RESULT\_TYPE a unique name \([here are existing result-types](https://github.com/LightTable/Clojure/blob/d9ea2e5dc7a1b5254544cbaaf5ba4b4802aa43e4/src/lt/plugins/clojure.cljs#L320-L367)\). The behavior's `:reaction` will get the eval result as the second argument. For an example behavior, see [statusbar :result-type](https://github.com/LightTable/Clojure/blob/d9ea2e5dc7a1b5254544cbaaf5ba4b4802aa43e4/src/lt/plugins/clojure.cljs#L331-L340).

2. Add the behavior's unique name to the :editor.clj tag in a behaviors file. For the statusbarexample, see [its behavior](https://github.com/LightTable/Clojure/blob/d9ea2e5dc7a1b5254544cbaaf5ba4b4802aa43e4/clojure.behaviors#L67).

3. Eval clojure code with the custom :result-type. An eval call should look [like this](https://github.com/LightTable/Clojure/blob/d9ea2e5dc7a1b5254544cbaaf5ba4b4802aa43e4/src/lt/plugins/clojure.cljs#L106-L111). For example, a behavior triggered by `:editor.eval.clj.result.mine` would have the following eval call:


`clojurescript(lt.object/raise lt.plugins.clojure/clj-lang :eval! {:origin editor :info (assoc (@editor :info) :meta {:result-type :mine} :code "(inc 41)")})`

### How do I read and write files?

Use `lt.objs.files/open-sync` and `lt.objs.files/save` respectively. For example:

`clojurescript(lt.objs.files/save "example.edn" "{:amazong true}")(lt.objs.files/open-sync "example.edn")`

### How do I copy to and paste from the system clipboard?

Use `lt.objs.platform/copy` and `lt.objs.platform/paste` respectively.

### How do I make an HTTP request?

LightTable ships with the [NodeJS http lib](http://nodejs.org/api/http.html). Here's an example fn that makes a GET request and calls a fn on the resulting body:

\`\`\`clojurescript\(defn GET \[url cb\] \(let \[body \(goog.string.StringBuffer. ""\) req \(.get \(js\/require "http"\) url \(fn \[resp\] \(.on resp "data" \(fn \[data\] \(.append body data\)\)\) \(.on resp "end" \(fn \[\] \(cb \(.toString body\)\)\)\)\)\)\] \(.on req "error" \(fn \[err\] \(println "Request" url "failed with:" \(.-message err\)\)\)\)\) \)

\(GET "[http:\/\/www.lighttable.com](http://www.lighttable.com)" \#\(println "BODY" %\)\)\`\`\`

### How do I suppress git noise created by compiled js in a plugin repo?

Treat a plugin's compiled js as a binary file with:

`bashecho "*_compiled.js -diff\n*_compiled.js.map -diff" >> .git/info/attributes`

Now anytime compiled js shows up in a diff or log you only see:

diff --git a\/clojure\_compiled.js b\/clojure\_compiled.js index e8eb4ef..9f73576 100644 Binary files a\/clojure\_compiled.js and b\/clojure\_compiled.js differ

To do this globally for git, see [this example](https://github.com/cldwalker/dotfiles/commit/49fd8145193db1074e5639fcd8baf25b5aee19ed).

### How do I add post :init behavior to any object?

When an object is created with `object/create`, object's are initialized with their `:init` and then behaviors that trigger off of `:object.instant` are called. For an example behavior, see the one for [:lt.objs.style\/styles](https://github.com/LightTable/LightTable/blob/407e8a9f2395474c2494cb332a37df35d7ed5196/src/lt/objs/style.cljs#L55-L75). This means you can customize any object so tread carefully ;\)

### How do I call a shell command?

Since you have node's libraries available, use [child\_process.exec](http://nodejs.org/api/child_process.html#child_process_child_process_exec_command_options_callback). Here's an example `ls`:

`clojurescript(.exec (js/require "child_process") "ls" (fn [err stdout stderr] (when (seq stdout) (println "STDOUT: " stdout)) (when (seq stderr) (println "STDERR: " stderr))))`

There's also `lt.objs.proc/exec` which wraps `child_process.exec` and is geared towards objects and behaviors.

### How do I reload a plugin's behaviors?

To try changes you make in a plugin, use interactive eval or save your changes and restart Light Table. The `App: Reload behaviors` command will not reload the plugin source as mentioned in[\#1042](https://github.com/LightTable/LightTable/issues/1042).

### How do I add a node package to a plugin?

In your plugin's root, install the dependency e.g. `npm install X --save`. Then torequire it in your plugin: `(def node-lib (js/require (lt.objs.plugins/local-module "PLUGIN_NAME" "NODE_LIB_NAME"))))`.

See the [JavaScript plugin for an example](https://github.com/LightTable/Javascript/blob/a00e352e6ae5b384333801b47802eea3a4f73d55/src/lt/plugins/js.cljs#L21).

### Can I use node packages within LightTable itself for my plugin?

It is not advised to use the node modules shipping with LightTable and should not be considered part of the official plugin API. You can of course still pull in the package you need for your plugin independently of LightTable.

### How do I add a clojurescript library to a plugin?

In your plugin's project.clj, add an entry to `:dependencies`. Note that ClojureScript libraries [are global](https://github.com/LightTable/LightTable/issues/1091). If you're running into issues loading your dependency, check other plugins to see if they're pulling in a conflicting version.

### I've added a clojurescript library to my plugin, but I get a `TypeError: Cannot read property 'X' of undefined` error. Now what?
LightTable loads your plugin js file during bootstrap, so you have two options:
- _ Restart LightTable - this should load your cljs dependency when your plugin is loaded_ 
- Manually eval your dependency in the LightTable UI client. No guarantees that this option will work, though.

