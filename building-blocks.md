# Building blocks

In this chapter we'll cover some typical basic building blocks you might need for your plugin\(s\).

## Adding items to the sidebar

The right sidebar in Light Table allows for adding items dynamically. By default you have a command-bar, docs search, clients bar and file navigation bar. To illustrate how you might add an sidebar item of your own, let's create a super simple module browser\(\/listing\).

```clojure
(ns lt.plugins.myplugin.modules 
  (:require [lt.object :as object] 
            [lt.objs.sidebar :as sidebar]  
            [lt.objs.command :as cmd]   
            [lt.util.dom :as dom]) 
  (:require-macros [lt.macros :refer [defui behavior]]))

(defui wrapper [this]                                                 // 1.
  [:div.modulebrowser.filter-list 
    [:ul.results "Replace me..."]])

(defui module-item-ui [module]
  [:li
    [:p module]])

(defui modules-ui [modules]                                           // 2.
  [:ul.results (map module-item-ui modules)])

(defn render-modules [this modules]                                   // 3.
  (let [container (object/->content this)
        results-ul (dom/$ :ul.results container)]                     // 4.
    (dom/replace-with results-ul (modules-ui modules))))              // 5.

(behavior ::show-modules                                              // 6.
          :triggers #{:show-modules}
          :reaction (fn [this] 
                      (object/raise sidebar/rightbar :toggle this)    // 7.
                      (render-modules this dummy-modules)))           // 8.


(object/object* ::modulebrowser 
                :tags #{:myplugin.modulebrowser} 
                :label "My Plugin module browser" 
                :order 2
                :behaviors [::show-modules]                           // 9.
                :init (fn [this] 
                        (wrapper this)))                              // 10.


(def modulebrowser-bar (object/create ::modulebrowser))               // 11.
(sidebar/add-item sidebar/rightbar modulebrowser-bar)                 // 12.

(cmd/command {:command :show-nsbrowser
              :desc "MyPlugin: Show module browser"
              :exec (fn []                                            // 13.
                      (object/raise modulebrowser-bar :show-modules))})

```

1. `defui` is a macro which helps produce html dom elements. Here we just create a wrapper div for our sidebar item
2. Much like in [hiccup](https://github.com/weavejester/hiccup) in `defui` we can work with markup as data structures. We can map, filter and the like. Here we map over the modules to produce `li` elements
3. We've created a function responsible for rendering our modules list
4. `dom/$` is a utility function for selecting the first item satisifying the given query selector. We retrieve the empty ul element 
5. Now we replace the empty ul element from above with the ul element created from `render-modules`
6. We've created a behavior to allow the display of the modulebrowser to be configurable
7. We raise a `:toggle` trigger on the `rightbar` object in the `lt.objs.sidebar` namespace. The responding behavior
  will toggle the display of the sidebar and making sure that our modulebrowser is shown as the active\/visible item \(alternatively hiding the sidebar all together\)
8. We render our current list of modules. Currently hardcoded, so imagine the list of modules is created somewhat more dynamically !
9. We've configured our modulebrowser object programtically to be tied to our show behavior. Normally you would do this in a `.behaviors` file for your plugin declaratively 
10. The init function is called upon creation of our object. If the init function returns markup, that will be available
  through the `object/->content` function we saw in our `render-modules` function.
11. We create an instance of our modulebrowser object
12. This is how we add an additional item to the Light Table sidebar. It will be added, but not visible \(until we trigger the `:toggle` behavior mentioned previously
13. We've created a command so that you are able to invoke display\/toggling of the module browser. When the command is invoked, it will trigger our `show-modules` behavior

> For a more advanced example you may want to look at this [blogpost](http://rundis.github.io/blog/2015/lt_react.html) for how you might use [React](https://facebook.github.io/react/) and [Quiescent](https://github.com/levand/quiescent) to render stuff in a sidebar.

## Adding an item to the statusbar

The statusbar is a convenient area for showing small snippets of useful information in Light Table. It is located at the bottom of UI in Light Table. In this example we are going to create a simple status item that shows the number of lines for the currently focused\/active editor.

```clojure
(ns lt.plugins.linecounter
  "Example statusbar item"
  (:require [lt.object :as object]
            [lt.objs.editor :as editor]
            [lt.objs.editor.pool :as pool]
            [lt.objs.statusbar :as statusbar]
            [crate.binding :refer [bound]])
  (:require-macros [lt.macros :refer [defui behavior]]))

(defui linecount-ui [{:keys [linecount]}]                                              // <1>
  [:span (str "Lines: " linecount)])

(behavior ::update-linecount                                                           // <2>
          :triggers #{:update!}
          :reaction (fn [this linecount]
                      (object/assoc-in! this [:linecount] linecount)))

(object/object* ::linecounter                                                          // <3>
                :triggers #{}
                :behaviors #{::update-linecount}
                :init (fn [this]
                        (statusbar/statusbar-item (bound this linecount-ui) "")))      // <4>

(def linecounter (object/create ::linecounter))                                        // <5>
(statusbar/add-statusbar-item linecounter)                                             // <6>

(behavior ::on-editor-linecount                                                        // <7>
          :triggers #{:focus! :change}                                                 // <8>
          :reaction (fn [ed]
                      (when (= ed (pool/last-active))                                  // <9>
                        (object/raise linecounter :update! (editor/line-count ed)))))

(object/tag-behaviors :editor [::on-editor-linecount])                                 // <10>


```

1. The Ui for our simple line counter, just the displays a label and the given line count
2. This behavior updates the :linecount entry in the linecounter object described in 3
3. We create an object to represent our line counter
4. When the object is initialized we create a statusbar-item element. `bound` is a macro that adds a watcher to the state of our object, whenever the state changes it will replace the content of our statusbar-item with whatever our linecount-ui function returns\). This way the view becomes "derived" from our object state at all times.
5. We create an instance of our linecounter object
6. Add the linecounter to the statusbar
7. We define an behavior to trigger for an editor object
8. The behavior should trigger whenever an editor receives focus or whenever the number of lines change. For simplicity we listen for change events, so that's any change which is a bit more frequent than we need.
9. Since we are listening to change events for any editor, we want to constrain it so that we don't update the line counter other than when the event is related to the currently active editor. We get trigger the ::update-linecount behavior and pass it the current line count for the editor in question
10. Rather than configuring the editor related behavior in a .behaviors file, we do it programaticcaly here by adding our behavior programatically to all objects with the `:editor` tag \(ie any editor\).

![](/assets/statusbar-item.png)
