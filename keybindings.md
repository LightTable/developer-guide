> ##### Work in Progress
> Editor's Note: Feel free to expand, relocate, or integrate this into the dev guide as you see fit.

# Keybindings

Check out the [keybinding section](https://github.com/LightTable/LightTable/blob/master/doc/behavior-and-keymap-configuration.md#keybindings) too for some basic information about keybindings. The following sections assumes you are familiar with the basic premises of keybindings... which are explained in the above link.

### Light Table Core

The default keybindings of Light Table can be found at `deploy/core/settings/default/default.keymap` and, barring any bugs or keybinding conflicts, should rarely be modified. This is because plugins can also add keybindings as they desire. If they map over a default keybinding, or the default keymap suddenly maps over their keybinding, it will cause confusion and frustration.

The user and plugin keymaps should overrule the default keymap in nearly all instances. There are currently no known reserved keybindings.   

### Plugin

At the highest directory in your plugin repo, you can create a file ending in `.keymap`... preferably named after your plugin. For instance, the file would be named `foo.keymap` if  your plugin is named `foo`. Doing so will let you define keybindings for your plugin.

In this `.keymap` file, which is simply EDN, you can add and remove keybindings. Here is a small example for the foo plugin:

```Clojure
{:+ 
 {:foo.focused { "o" [:lt.plugins.foo/open-selection]
                 "x" [:lt.plugins.foo/close-parent]}}
 :-
 {:editor { "pmeta-s" [:save]}}
 :+
 {:editor { "pmeta-s" [:lt.plugins.foo/open-selection]}}}
``` 

In this example there are several things to note:

- Key-value pairs with the key `:+` will be added to the overall keymap.
- The `:-` will remove a keybinding from the keymap. Avoid doing this unless you have good reason (and the user is aware). 
- The command associated with the keybinding being added or removed does not even have to be found in the plugin.