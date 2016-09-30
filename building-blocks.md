# Building blocks

In this chapter we'll cover some typical basic building blocks you might need for your plugin\(s\).





## Adding items to the sidebar
The right sidebar in Light Table allows for adding items dynamically. By default you have a command-bar, docs search, clients bar and file navigation bar. To illustrate how you might add an sidebar item of your own, let's create a super simple module browser(/listing).


```clojure
(ns lt.plugins.myplugin.modules 
  (:require [lt.object :as object] 
            [lt.objs.sidebar :as sidebar]  
            [lt.objs.command :as cmd]   
            [lt.util.dom :as dom]) 
  (:require-macros [lt.macros :refer [defui behavior]]))

(defui wrapper [this]                                                  // 1.
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
1. defui is a macro which helps produce html dom elements. Here we just create a wrapper div for our sidebar item
2. Much like in [hiccup](https://github.com/weavejester/hiccup) in `defui` we can work with markup as data structures. We can map, filter and the like. Here we map over the modules to produce `li` elements
3. We've created a function responsible for rendering our modules list
4. `dom/$` is a utility function for selecting the first item satisifying the given query selector. We retrieve the empty ul element 
5. Now we replace the empty ul element from above with the ul element created from `render-modules`
6. We've created a behavior to allow the display of the modulebrowser to be configurable
7. We raise a `:toggle` trigger on the `rightbar` object in the `lt.objs.sidebar` namespace. The responding behavior
will toggle the display of the sidebar and making sure that our modulebrowser is shown as the active/visible item (alternatively hiding the sidebar all together)
8. We render our current list of modules. Currently hardcoded, so imagine the list of modules is created somewhat more dynamically !
9. We've configured our modulebrowser object programtically to be tied to our show behavior. Normally you would do this in a `.behaviors` file for your plugin declaratively 
10. The init function is called upon creation of our object. If the init function returns markup, that will be available
through the `object/->content` function we saw in our `render-modules` function.
11. We create an instance of our modulebrowser object
12. This is how we add an additional item to the Light Table sidebar. It will be added, but not visible (until we trigger the `:toggle` behavior mentioned previously
13. We've created a command so that you are able to invoke display/toggling of the module browser. When the command is invoked, it will trigger our `show-modules` behavior




> For a more advanced example you may want to look at this [blogpost](http://rundis.github.io/blog/2015/lt_react.html) for how you might use [React](https://facebook.github.io/react/) and [Quiescent](https://github.com/levand/quiescent) to render stuff in a sidebar. 

