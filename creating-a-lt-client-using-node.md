# Creating a LT Client using Node
Typically when you create a Language plugin for Light Table you will find examples on how to do that for a TCP/IP client. This allows you to code the client in whatever language you prefer (perhaps in the language that your plugin is to support). However sometimes it might be more convenient to code the client using Node as the container for the external process. As it happens Light Table ships with Node (through Electron) so why not make use of it ? The [elm-light plugin] (https://github.com/rundis/elm-light) uses this approach to provide language support for [Elm](http://elm-lang.org/)

## Convention
First of we'll establish some conventions on how you might structure your plugin

### Directory layout
* $plugindir - The root directory of your plugin * node - In this directory you place the node js code for your plugin * node_modules - If your plugin makes use of node modules currently provided as part of the LT distribution, you can place them here * src - This is where you place the ClojureScript code for your plugin

You might decide there are other directories (css, codemirror addon etc) that you wish to add as well, but for the purposes of this tutorial we'll just assume the above directory structure.

### Other conventions used
We'll refer to our target language as *mylang* from here on. So when you see mylang, just replace with whatever language you would be targeting.

## The Node client
We'll start of with creating a file `$plugindir/node/mylang-client.js`


```javascript 
// Add any required node modules you might need
var fs = require("fs"); 
var cp = require("child_process");
var custom = require("some_custom_node_plugin"); // some node module you might need that you bundle with your plugin

var myLangGlobals = {}; // maybe you need some global state for your plugin

/**
 * Bootstrap the client**/
startRepl(
 function(err) {
   console.error("Failed to start repl for mylang: " + err);
   handleClose();
 },
 function(out) {
   console.out("Repl started ok");
   startMessageListener();
  },
  {execPath: process.execPath, cwd: process.cwd()}
);

/**
 * Send acc to LT and start listening for messages
**/
function startMessageListener() {
  send([1, "mylang.client.connected", []]); // notify lt we`re ready to receive messages

  process.on("message", function (msg) {
  var cb = msg.cb; // id for object that should receive response message
  var cmd = msg.command; // command to be executed
  var data = msg.data; // The payload for the command

  try {
    switch (cmd) {
      // You must handle this ! This message is automatically sent by LT when disconnecting manually (or terminating LT)       
      case "client.close": 
        handleClose();
        break; 
      case "editor.eval.elm": 
        handleEval(cb, data);
        break;
      }
    } catch (ex) {
      console.error("Error in mylang client message listener for command: " + cmd);
      console.error(ex);
      handleClose();
    }
  });
}

/**
 * Example function to start a long running process that your client holds on to
**/
function startRepl(err, success, params) {
  myLangGlobals.repl = cp.spawn("mylang-repl", [params.execPath], {cwd: params.cwd});
  myLangGlobals.repl.stdout.on("data", function (out) {
    // whatever, some logic to check if the process started ok
    success(someOkMsg);
  });
  myLangGlobals.repl.stdout.on("data", function (err) {
  // whatever, some logic to check if the process failed somehow 
    err(someErr);
  });
}

/**
 * Close/shutdown the client process (and cleanup any forked processes etc
**/
function handleClose() {
  console.log("Exiting...");
  process.exit(0);
}

/**
* Handle eval function
**/
function handleEval(cbId, data) {
  // ... do whatever needed to eval  send([cbId, "editor.mylang.eval.res", someEvalRes]);
}

// Emit message back to LT
function send(msg) { process.send(msg); }
```

This might not be idiomatic JavaScript code, but hopefully it's clear enough to get you started on creating a client.

# Communicating with the client from LT
Let's create a separate ClojureScript namespace for handling communication with the Node client:`$plugindir/src/lt/plugins/mylang/clients.cljs`

## Basic setup

```Clojure
(ns lt.plugins.mylang.clients
  (:require [lt.objs.files :as files]
            [lt.object :as object]
            [lt.objs.console :as console]
            [lt.objs.notifos :as notifos]
            [lt.objs.clients :as cs]
            [lt.objs.proc :as proc]
            [lt.objs.plugins :as plugins])
  (:require-macros [lt.macros :refer [behavior]]))

(def cp (js/require "child_process"))
(def os (js/require "os"))
(def mylang-client-path (files/join (plugins/find-plugin "mylang") "node/mylang-client.js"))
(def mylang-node-path (files/join u/mylang-plugin-dir "node_modules"))
```

We require a couple of std node modules which we will use soon. Also note the usage of plugins/find-plugin which locates the path to our plugin.

## Forking the node client process

```Clojure

(defn on-elm-message [client data]
  (let [msg (js->clj data :keywordize-keys true)] ; 1 - convert message into a clojure sequence (cbId, behavior, payload)
    (if (= (second msg) "elm.client.connected")
      (do 
        (object/raise cs/clients :connect client) ; 2 - Notifies LT that the client is connected. Updates connect bar.     
        (object/raise client :connect)) ;           3 - Finalizes connection process and sends any messages queued
        (object/raise cs/clients :message msg)))) ; 4 - Triggers routing of the message to intended behaviour


(defn start-mylang-worker [path client]
  (let [worker (.fork cp ; 1 - Fork a new node process
                      mylang-client-path #js [] ; 2 - Add args if needed
                      (clj->js {:execPath (.-executable js/process) ; 3 - Use electron node exe
                                :cwd path ; 4 - Working dir 
                                :silent true
                                :env (if (= (.platform os) "win32") ; 5 - Add env needed 
                     {:NODE_PATH mylang-node-path}
                     (proc/merge-env {:NODE_PATH mylang-node-path}))}))]
    (.on (.-stdout worker) "data" #(console/log (str "out: " msg)))
    (.on (.-stderr worker) "data" (console/error (str "Error from runner: " err)))
    (.on worker "message" #(on-elm-message client %))
    (.on worker "exit" #(cs/rem! client)) ; 6 - 

worker))
``` 
* `(.-executable js/process)` - Uses the node executable shipped with LT (through Electron)
* We set the NODE_PATH environment path to be able to use our custom node modules easily. 
* Here we have made a distinction with default env params between windows and other platforms. `proc/merge-env` allows us to use LT's default behaviour for picking up environment variables like PATH etc. On windows this might not be necessary for your case.

## Making a connection

```Clojure
(behavior ::send!
          :triggers #{:send!}
          :reaction (fn [client msg]
                      (.send (:worker @client) (clj->js msg)))) ; Sends a message to client using node ipc

(defn start-elm-client [{:keys [proj-path client]}]
  (notifos/working "Connecting..")
  (let [worker (start-mylang-worker proj-path client)]
    (object/merge! client
                   {:name (files/basename proj-path)
                    :dir proj-path ; 1 - Directory is used by LT to locate an appropriate client
                    :worker worker ; 2 - We let the client object(atom) hold on to our client proc 
                    ;; 3 - Set of commands our client supports. Also used to locate client candidates
                    :commands #{:editor.mylang.eval}}) 
    (object/add-behavior! client ::send!))) ; 4 - Let LT know what behaviour handles sending messages

; This function serves as the entry point for connecting to a new client
(defn try-connect [{:keys [info]}]
  (let [path (:path info) ; 1 - Info could typically be the info key from and editor instance of mylang
        proj-path (u/project-path path) ; 2 - Some util method to derive root-path for your mylang project
        client (cs/client! :elm-client)] ; 3 - Creates a client object instance (atom)

    (if proj-path
      (start-mylang-client {:proj-path proj-path ; 4 - Start the client process
                           :client client})
      (do ; 5 - Notify user of missing precondition. Clean up/remove client instance
        (notifos/done-working)
        (notifos/set-msg! (str "Couldn't find a mylang.project in any parent of path: " path) {:class "error"})
        (cs/rem! client)))
  client))
```

# Using the connection for the eval behaviour

```Clojure
(ns lt.plugins.mylang.core
  (:require [lt.plugins.mylang.clients :refer [try-connect]]
            [lt.object :as object] 
            [lt.objs.editor :as editor]
            [lt.objs.notifos :as notifos]
            [lt.objs.console :as console]
            [lt.objs.clients :as clients]
            [lt.objs.eval :as eval])
  (:require-macros [lt.macros :refer [behavior]]))

; 1 - Behavior wired to an mylang editor for doing eval of a selection of code
(behavior ::on-eval.one
          :desc "Mylang repl: Eval current selection"
          :triggers #{:eval.one}
          :reaction (fn [ed]
                      (let [pos (editor/->cursor ed)
                            ;; Pick out code and meta data for eval 
                            info (conj (:info @ed) 
                                       {:code (editor/selection ed) 
                                        :meta {:start (-> (editor/->cursor ed "start") :line)
                                        :end (-> (editor/->cursor ed "end") :line)}})]
    
                        ;; Raise generic eval behavior. 
                        ;; `:origin` lets LT know where route result message (rf cb id mentioned earlier) 
                        (object/raise mylang :eval! {:origin ed :info info}))))

;; Generic eval behavior
(behavior ::eval!
          :triggers #{:eval!}
          :reaction (fn [this event]
                      (let [{:keys [info origin]} event]
                        (notifos/working "Evaluating mylang...") 
                        ;; 1. `eval/get-client` will try to locate a suitable client, 
                        ;; if one doesn't exist it will connect to one using or try-connect function
                        ;; 2. `clients/send` will create a message and either queue the message (if not yet connected)
                        ;; or trigger the send! behavior we previously defined
                        ;; 3. `:only origin` will tell LT to only route the response message back to this particular editor
                        ;; object. The id of the editor object instance is what's eventually sent to our node client
                        (clients/send 
                          (eval/get-client! {:command :editor.eval.mylang
                                             :origin origin
                                             :info info
                                             :create try-connect})
                          :editor.eval.mylang info
                          :only origin))))

;; When our node client sends a response message for the eval command, this is the behavior that
;; is eventually triggered. Here we delegate the displaying of results to a default behaviour in LT for showing
;; inline editor results 
(behavior ::eval-result
          :desc "Mylang repl: Eval result"
          :triggers #{:editor.mylang.eval.res}
          :reaction (fn [ed res]
                      (notifos/done-working "Mylang evaluated")
                      (object/raise ed
                                    :editor.result
                                    (:result res)
                                    {:line (-> res :meta :start)})))

(object/object* ::mylang-lang
                :tags #{:mylang.lang})

;; Object(/atom) we may use for holding onto generic stuff for our language plugin. 
;; We may also wire behaviours to this object
(def mylang (object/create ::mylang-lang))
```

## Summary
We've covered quite a bit of ground here. The most important bits we tried to communicate is that it isn't all that difficult to create a language client using node. This might hopefully serve as a template for future language plugins where using a node client might be a good fit. For the elm plugin it certainly was a good match, for other languages youmight prefer to code the client in the language you target. Then you should use the tcp/ip approach
