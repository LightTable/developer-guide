# Light Table ClojureScript tutorial

This tutorial is based on David Nolans [Light Table ClojureScript tutorial](https://github.com/swannodette/lt-cljs-tutorial)

The minimum fuzz way to get it up and running is by using the User plugin as a  starting point. So follow these steps to get going:

* Add the Light Table User plugin to your workspace with the command `Settings: Add User plugin to workspace`
* Open the `user.cljs` file by using the command `Settings: User script`
* Place the cursor anywhere in the file and save \(cmd-s\/ctrl-s\) - This will connect the user project so that we have an environment to evaluate ClojureScript in
* Okay now open a new editor using the command: `File: New File`
* Set the syntax of the editor to be ClojureScript by using the command: `Editor: Set current Editor Syntax` and then search and select `ClojureScript`
* Now copy the contents from the [tutorial file \(lt-cljs-tutorial.cljs\)](https://raw.githubusercontent.com/swannodette/lt-cljs-tutorial/master/lt-cljs-tutorial.cljs) into your empty ClojureScript editor 
* Scroll to the top of the file and place the cursor inside or next to the ns declaration and invoke the command `Eval: Eval a form in the editor` \(cmd-enter\/ctrl-enter\)
* You will be asked to provide a project connection. Select `Light Table UI`
* Now you should be able to follow along with the instructions in the tutorial and evaluate the examples as you go.

![](/assets/lt-tutorial.png)


> You should obviosly check out the official ClojureScript resources at [clojurescript.org](https://clojurescript.org/). B**ut please be aware that Light Table is currently using a very old version of ClojureScript.** That does not mean you can't use a newer version of ClojureScript when working with projects, but it means that when it comes to developing plugins and working on Light Table Core you have to take this into consideration !

