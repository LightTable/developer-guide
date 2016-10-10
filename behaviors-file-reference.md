# Behaviors File Reference

## Loading JavaScript

`[:app :lt.objs.plugins/load-js "myplugin_compiled.js"] ; This is the compiled js for your plugin`

or multiple \(in order\)

`[:app :lt.objs.plugins/load-js ["myplugin_compiled.js" "js/external.js"]]`

## Loading CSS

`[:app :lt.objs.plugins/load-css "css/myplugin.css"]`

## Loading a keymap

`[:app :lt.objs.plugins/load-keymap "myplugin.keymap"]`

## Associate new file types

`[:files :lt.objs.files/file-types [{:exts [:elm], :mime "text/x-elm", :tags [:editor.elm], :name "elm"}]]`

Light Table ships with a wide range of mapping to file types to provide syntax highlighting. However you might be developing a plugin for a new language not covered. This config allows you to associate a new file extensions and provide a tag for editors of this file type. In the example above we've added support for files from the Elm programming language. Every Elm file editor object will now get a tag `:editor.elm`.This is obviously useful when you want to configure behaviors that are specific for Elm editors \(say like code eval, docs etc\). 

## Adding tags to objects of a given tag

`[:editor.elm :lt.object/add-tag :docable] ;; this tag says that Elm editors supports behaviors for showing language docs`

You can add additional tags to an object of a given tag by using `:lt.object/add-tab`. This is sometimes useful to allow more flexible configuration of objects and behaviors.  

## Providing skins and themes

` [:app :lt.objs.style/provide-skin "superduper" "css/skins/superduper.css"]````[:app :lt.objs.style/provide-theme "super-light" "css/themes/super-light.css"]`

* Skins override the std look and feel of all common Light Table elements \(sidebar, doc popups, workspace tree etc\)
* Themes provides overrides of the look and feel of editors in Light Table



