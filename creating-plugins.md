# Creating plugins

## Getting started

If you are not already familiar with the [BOT Architecture](/the-light-table-bot.md) in Light Table, it's highly recommended that you read through that before starting to develop a plugin for Light Table.

### Bootstrap using the Leiningen lt-plugin

The easiest route to get started with creating a new plugin is to use the `lt-plugin` for [Leiningen. ](http://leiningen.org/) If haven't installed Leiningen and don't feel like doing so, you can obviously create the files listed below manually.

To get started. Just type the following in a terminal 
- `#> lein new lt-plugin myplugin`
- Then add the <myplugin> folder to your Light Table workspace

|Structure      | Description |
| ------------- |-------------|
|![](/assets/plugin-files.png) | <ul><li><b>myplugin.cljs</b> - Source entry point for your plugin.</li><li><b>myplugin.behaviors</b> - Configuration of your plugin. Here you can configure behaviors, load the compiled plugin source, load additional JavaScript or CSS files etc.</li><li><b>myplugin_compiled.js</b> - This is just a placeholder for the compiled source of your plugin. It will be generated/updated by the compiler</li><li><b>plugin.edn</b> - Plugin description which described the key attributes of your plugin. It is necessary for the plugin manager in Light Table</li><li><b>project.clj</b> - ClojureScript - Project file for your plugin. It is needed by the compiler. Here you can add 3.rd party ClojureScript libraries</li></ul>|

> For a more in-depth explanation of what you can configure in your `.behaviors` file check out the [Behaviors File Reference](/behaviors-file-reference.md)


## Developing

You might find inspiration from the following plugin creation guides

* [Creating a Skin](/creating-a-skin.md)
* &lt;Need an example of something in between...&gt;
* [Creating a Language plugin](/creating-a-language-plugin.md)

## Submit a plugin

If it's your first time submitting a plugin, make sure you have a valid `plugin.edn`. Fill out the keys as follows:

* `:name`\*: Camel case is recommended
* `:version`\*: Must be in format X.X.X e.g. 0.1.4
* `:source`\*: Must be a github url e.g. https:\/\/github.com\/USER\/PLUGIN
* `:behaviors`\*: Refers to the behaviors file which is needed to load the plugin and define any keybinding or behavior.
* `:desc`: Primary means users can find your plugin
* `:author`: Your name and another useful way to find your plugin

`*` Means the field is required.

When releasing a new version, you must update the `:version` key and push the new git tag e.g. `git tag X.X.X && git push --tags`. With your updated `plugin.edn`, add or update your plugin to the [plugin metadata repository](https://github.com/LightTable/plugin-metadata) using [these instructions](https://github.com/LightTable/plugin-metadata#submit-a-plugin). 

