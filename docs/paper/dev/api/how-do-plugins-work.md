---
slug: /dev/how-do-plugins-work
description: How plugins work in Paper.
---

# How Plugins Work

## Introduction

Plugins are a way to extend the functionality of a Minecraft server. They are written in JVM-based languages such as 
Java, Kotlin, Groovy or Scala. Plugins are loaded from the `plugins` folder in the server directory. Plugins will be 
loaded from a `.jar` file.  Each plugin has a main class that is specified in the plugin's `plugin.yml` file. This
class must extend JavaPlugin, and is the entry point for the plugin and is where the plugin's lifecycle methods are 
defined.

:::warning

We do not recommend writing code inside your main class's constructor as there are no guarantees about what
API is available at that point. Instead, you should use the `onLoad` method to initialize your plugin. Also,
do not call your plugin's constructor directly. This will cause issues with your plugin.

:::

## Plugin Lifecycle

Plugins are loaded and unloaded at runtime. When a plugin is loaded, it is initialized and enabled. When a plugin is 
unloaded, it is disabled and finalized.

### Initialization

When a plugin is loaded, it is initialized. This means that the plugin is loaded into memory and its `onLoad` 
method is called. This method is used to initialize the plugin and set up any resources that it needs. Most of the
Bukkit API is not available at this point, so it is not safe to interact with it.

### Enabling

When a plugin is enabled, its `onEnable` method is called. This method is used to set up any resources that the plugin 
needs to run. This method is called when the plugin is initialized but before the server has started ticking, so it is 
safe to register event listeners and other resources that the plugin needs to run, however often not safe to interact 
with a lot of APIs.

This is when you can also open database connections, start threads, and other things that are not safe to do in the
`onLoad` method.

### Disabling

When a plugin is disabled, its `onDisable` method is called. This method is used to clean up any resources that the 
plugin has allocated. This method is called before all plugins are unloaded, and is meant for any cleanup that needs to
be done before the plugin is unloaded. This may include saving data to disk or closing connections to databases.

## Event Listeners

Events are a way for plugins to listen to things that happen in the server and run code when they are fired. For 
example, the `PlayerJoinEvent` is fired when a player joins the server. This is a more performant way to run code when 
something happens, as opposed to constantly checking. See our [event listener page](/paper/dev/event-listeners) for more.

Some events are cancellable. This means that when the event is fired, it can be cancelled which negates or stops the 
effect of the event. For example, the `PlayerMoveEvent` is cancellable. This means that when it is cancelled, the player 
will not move. This is useful for things like anti-cheat, where you want to cancel the event if the player is moving too fast.

It is important to think about how "hot" an event is when writing event listeners. A "hot" event is an event that is fired
very often. For example, the `PlayerMoveEvent` is fired every time a player moves. This means that if you have a lot of
expensive code in your event listener, it will be run every time a player moves. This can cause a lot of lag. It is
important to keep event listeners as lightweight as possible. One possible way is to quickly check if the event should
be handled, and if not, return. For example, if you only want to handle the event if the player is moving from one block
to another, you can check if the player's location has changed blocks. If it hasn't, you can return from the listener.

## Commands

Commands are a way for players, the console, RCON and command blocks to run code on the server. Commands are registered
by plugins and can be run by command senders. For example, the `/help` command is registered by the server and can be
run by players. Commands can be run by players by typing them in the chat or by running them from a command block.

Commands can have arguments. For example, the `/give` command takes an argument for the player to give the item to and
an argument for the item to give. Arguments are separated by spaces. For example, the command `/give Notch diamond` will
give the player named Notch a diamond. Note here that the arguments are `["Notch", "diamond"]`.

### Permissions

Permissions are a way to control who can run commands and who can listen to events. Permissions 
are registered by plugins and can be checked by other plugins. Permissions can be granted to players and groups. 
Permissions have a hierarchical nature. For example, the `paper.command.help` permission is a sub-permission to 
`paper.command`. This means that if a player has the `paper.command` permission, they will also have the 
`paper.command.help` permission.

## Configuration

Plugins can have configuration files. These files are used to store data that the plugin needs to run. For example, a
plugin that adds a new block to the game might have a configuration file that stores the block's ID. Configuration files
are stored in the `plugins` folder in the server directory. Configuration files are written in YAML. See 
[here](/paper/dev/plugin-configurations) for more information.

## Scheduling Tasks

Plugins can schedule tasks to run at a later time. This is useful for things like running code after a certain amount
of time has passed. For example, a plugin might want to run code after 5 seconds. This can be done by scheduling a task
to run after 100 ticks - one second is 20 ticks during normal operation. It is important to note that tasks might be 
delayed if the server is lagging. For example, if the server is only running at 10 ticks per second, a task that is 
scheduled to run after 100 ticks will take 10 seconds.

In Java, typically you could use `Thread.sleep()` to delay the execution of code. However, if the code is running on the main
thread, this will cause the server to pause for the delay. Instead, you should use the `Scheduler` API to schedule tasks
to run later. Learn more about the `Scheduler` API [here](/paper/dev/scheduler).

## Components

Since Minecraft 1.7 and the introduction of the `Component` API, plugins can now send messages to players that contain
rich text. This means that plugins can send messages that contain things like colors, bold text, and clickable links.
Colors were always possible, but only through the use of legacy color codes.

Paper implements a library called `Adventure` that makes it easy to create and send messages to players. Learn more
about the `Adventure` API [here](https://docs.advntr.dev/) from their docs or our docs 
[here](/paper/dev/component-api/introduction).
