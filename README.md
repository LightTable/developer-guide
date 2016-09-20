# Light Table Developers Guide

**Work in progress ...**

Welcome to the Light Table Developers Guide. This guide has two key purposes;

* Guide you on the what and hows for developing a Light Table plugin
* Provide the instructions necessary for you if you wish to help out with contributing to the development of the core of Light Table

> A plugin is the key mechanism in Light Table for customizing and extending its behaviours and features. The plugin system in Light Table is one of it's biggest strengths and it allows you to virtually infinitely customize or extend Light Table. Plugins allow you to do everything from just overriding some setting to almost completely changing the entire behaviour of Light Table.  Through plugins you can implement things like; Skins, editor features, simple or advanced language support etc. Actually all your personal user settings is even implemented as a User plugin that is automatically set up for you when you install Light Table.

## A quick technical overview

Almost the entire core of Light Table is implemented using [ClojureScript](http://clojurescript.org/). ClojureScript is a functional language that compiles down to JavaScript. It may be unfamiliar to you, but don't let that scare you off. It really is a super powerful language which you'll come to love once you put in the initial investment to get you up and running. You can check out the [Light Table ClojureScript Tutorial](/light-table-clojurescript-tutorial.md) to get you quickly started.

Light Table is based on two key components:

* [Electron](http://electron.atom.io/) - A framework for creating native Desktop web applications. It comes with two vital parts a node server and a chromium browser. 
* [CodeMirror](https://codemirror.net/) - An editor implemented in JavaScript for use in Web Browsers. The majority of the editor features in Light Table is based on CodeMirror and CodeMirror add-ons. 

This a very powerful platform which allows you to do some really awesome things. Anything you can do in a browser you can wield Light Table to do. Couple that with the power of having a Node.js server and the npm package eco-system at your disposal, there is really a lot of power at your disposal.



If you are new to Light Table, let's move on to the [Getting Started chapter](/chapter1.md) !

