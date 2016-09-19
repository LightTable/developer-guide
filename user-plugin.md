# User plugin

The simplest way to get started developing something for Light Table is to play with the User plugin. In this chapter
we're going to walk through a couple of simple examples.

## Location

The location of the user plugin depends on your OS

* Mac: ~\/Library\/Application\ Support\/LightTable\/User
* Linux: ~\/.config\/LightTable\/User
* Windows: %APPDATALOCAL%\/LightTable\/User

## Getting started

Your complete configuration, including plugins you've installed, is stored in a User plugin. Since the User plugin is just a directory, you can share it by putting it under revision control e.g. git and uploading it to a service like Github. To explore it

* add it to your workspace with the command `Settings: Add User plugin to workspace`
* Any custom keybindings and behaviors are added to `user.keymap` and `user.behaviors`. 
* To write commands, behaviors and more, see `src/lt/plugins/user.cljs`. 
* To open your `user.cljs` at anytime use the command `Settings: User script`. 
* Inside the default `user.cljs` is an example command and behavior.  

Run the command `User: Say Hello` to see your own command!

## Adding your own command

Commands are the user facing functionality in Light Table. They are typically listed in the Command Pane and you can assign shortcuts to the. The are the easiest place to start, so let's go ahead and make our own little command. Open `user.cljs` and add this snippet.

```clojure
(cmd/command {:command :user.print-filename                       ;; 1.
              :desc "User: Print current file name"               ;; 2.
              :exec (fn []                                        ;; 3.
                      (let [ed (lt.objs.editor.pool/last-active)] ;; 4.
                        (println (-> @ed :info :path))))})        ;; 5.

```

1. We give our command an id. That way we can refer to that command from other places. Typically you want to be able to assign keyboard shortcuts to a command. The id is how you can map a keyboard shortcut to your command.
2. Give the command a sound description. Commands are shown in the Command Pane in Light Table, so you want to provide a meaningful description so people know what the command does. 
3. This is where we define what's going to happen when the command is to be executed
4. We are using a function `last-active` from the `lt.objs.editor.pool` namespace to get hold of the current Editor. Of currently focused open file if that makes more sense. Actually we are not getting the file, but we are getting an `object` representing that file. 
5. The editor object we have access to is a special kind of object. It's basically a wrapped ClojureScript [atom. ](http://clojure.org/reference/atoms). Now that's not to important, the main point is that this object contains meta information that we can access. So in JavaScript terms you could read this as `ed.info.path`

**To make this command available we need to connect to the user plugin project and then evaluate it.**

1. Save the file - When you hit save it will first connect to the project and then compile the ClojureScript code in your user plugin to JavaScript. So first time it take a little time, after that it will be much faster.
2. With the cursor inside the command definition\/\(form in Clojure terms\) 
  1. Open the command pane \(ctrl-space\)
  2. Search for the command `Eval: Eval a form in editor`. Select that
  3. Light Table will tell you to connect to plugin. Click the button `Connect a Client`
  4. From the list of available clients select `Light Table UI`
  5. You should see `nil` shown after the form. If not try to evaluate one more time.


> When you evaluate code like we just did, LIght Table will during runtime compile the code \(using the ClojureScript compiler\) to JavaScript and add the resulting JavaScript to the running Light Table. You can now make changes to the command as you wish, and re-evaluate it. That will replace the runtime definition of the command with a new definition. This way of developing provides really fast turnaround and promotes a stepwise interactive approach to buidling things.

You should now be able to call the command. Just open the command pane and search for the description you gave the command. Select \`View -&gt; Console\` from the menu to open the console if you don't already have it open. Voila you should see the filename of the editor you had focus on when invoking your new command !

## Adding your own behavior

```clojure
(behavior ::user.display-filename                                           ;; 1.
          :desc "Show filename in statusbar for while when focused"         ;; 2.
          :triggers #{:focus}                                               ;; 3.
          :reaction (fn [ed]                                                ;; 4.
                      (lt.objs.notifos/set-msg! (-> @ed :info :path))))     ;; 5.
```

1. We give our behaviour an id. We are going to need this id to be able to configure when this behavior should be triggered
2. This is optional, but it's nice for documentation and you'll see that it's also helpful when we wire it up in the next section
3. Triggers are the definition of event name\(s\) that should trigger this behavior to execute
4. The reaction, defines the function that should be executed when the behavior is triggered. You'll also notice we've 
  assumed a parameter ed, ie we are expecting this behavior function to be called with an editor object. 
5. Finally we use the `set-msg!` function from the `lt.objs.notifos` namespace to set a message with the editors file name in the bottom status bar in Light Table. \(The message will by default be displayed for 10 seconds\)

**Assuming you have already gone through the command example in the previous example you need to:**

1. First scroll to the top of the `User.cljs` file and put the cursor inside the ns form and then evaluate that.
2. Then scroll back to the behaviour and evaluate that

**We are not quite done yet, we also need to tell Light Table how and what should make this behavior actually trigger**

1. Open the command pane and search for `Settings: User behaviors` and select that
2. Add the below definition to the file and then save

```clojure
[:editor :lt.plugins.user/user.display-filename]
```

* If you put your cursor over inside or right next to that line, you will see the description we provided to our behavior is shown highlighted next to the right bracket
* `:editor` is a tag that tells Light Table that this the behavior should be considered for any object in Light Table 
  that has this tag should be considered a candidate for it to be triggered. All open files are represented as editor objects and they all have by default this tag.
* The final piece of the puzzle is the trigger we defined for our behavior. So whenever someone\/somewhere raises a `:trigger` event on an editor instance Light Table now knows enough to invoke our behavior reaction function. Triggering :focus is something that is done inside another behavior in Light Table, let's not worry to much about that for now.

### Flexibility through the BOT architecture 

The combination of **B**ehavior, **O**bject and **T**ags is what is described as the [BOT Architecture](/the-light-table-bot.md) in Light Table. It gives you flexibility in several dimensions;

1. **Tags**: You can target behaviors to object with certain tags. So if you change the above behavior wiring to 
  1. `[:editor.javascript :lt.plugins.user/user.display-filename]`
  2. Now the behaviour will only trigger when JavaScript files\(\/editors\) receive the `:focus` trigger. BTW, you can have multiple behavior wirings for the same behavior, so if you would like this behavior to happen for Clojure and Elm files just add a line for each of them. 
  3. If you'd like to turn off the behavior you can just use the following syntax `[:editor :-lt.plugins.user/user-display-filename]`. Note the minus sigh. 
  4. When you save the behaviors file, the changes will be applied at once. Runtime.

2. **Triggers**: You can have multiple triggers for your behavior. Say you wanted to also display the filename whenever the editor changes. Just add `:change` to the set of triggers in your behavior and reevaluate it.

3. **Objects**: You can add or remove tags to objects both at declaration time, programatically or in your User.behaviors file. For example if you add the following to your `User.behaviors` file `[:editor :lt.obj/add-tag :mytag]` all editor objects in Light Table will also receive the `:mytag` tag.
4. **Behaviors**: What's more you can also add or remove behaviors to objects during runtime both in your  `User.behaviors`file or  programatically.  

You don't need to understand or use all this from the off, but hopefully you get and idea of the flexibility of the BOT architecture.

> You can get far by just using commands and normal ClojureScript functions, but if you want to create a runtime configurable plugin you'll want to start looking at using behaviors.

