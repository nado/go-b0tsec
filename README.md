![Go Version](https://img.shields.io/badge/go-1.5-brightgreen.svg)
[![Travis](https://travis-ci.org/Depado/go-b0tsec.svg)](https://travis-ci.org/Depado/go-b0tsec)
[![Coverage Status](https://coveralls.io/repos/Depado/go-b0tsec/badge.svg?branch=master&service=github)](https://coveralls.io/github/Depado/go-b0tsec?branch=master)
[![Go Report Card](http://goreportcard.com/badge/Depado/go-b0tsec)](http://goreportcard.com/report/Depado/go-b0tsec)

# Go-b0tsec

IRC bot written in Go with plugins and middlewares.

 - [Configuration](#configuration)
 - [Plugins](#plugins)
 - [Middlewares](#middlewares)
 - [Mixins](#mixins)
 - [Contributing](#contributing)
 - [License](#license)

## Configuration

This bot is intended to be simple to configure and as such, will use a yaml configuration file. Some values are needed to use some plugins or middlewares.

```yaml
# Bot Connection
bot_name:     awesomebot
server:       irc.freenode.net:6667
channel:      "#awesomechan"
tls:          false
insecure_tls: false
# Other Behavior
youtube_key: "YourVeryOwnYoutubeAPIKey"
plugins:
    - ud
    - ddg
    - anon
    - markov
    - karma
    - help
    - dice
    - afk
    - seen
middlewares:
    - logger
    - github
    - markov
    - afk
    - seen
    - youtube
    - title
```

 - `bot_name` : The name of your bot as it will appear on the channel.
 - `server` : The server on which you want to connect. Port is mandatory.
 - `channel` : The channel the bot should join when connected to the server.
 - `tls` : Whether or not to use TLS when connecting to the server.
 - `insecure_tls` : Ignore TLS errors (self signed certificate for example)
 - `youtube_key` : Your own google API key to fetch the Youtube API.
 - `plugins` : Names of the plugins you want to activate.
 - `middlewares` : Names of the middlewares you want to activate.

**Note : If you don't want to use Youtube capabilities or don't want to create an API key, then make sure to disable the `youtube` middleware**

**Update : The external resource collection has now moved to a different repository. See [Depado/periodic-file-fetcher](https://github.com/Depado/periodic-file-fetcher) to use this within your bot.**

## Plugins

A plugin is a command, which can be called using the "!command" syntax. It's the most usual thing you'll ever see with IRC bots.
To create your own plugin, take a look at the [plugins/example/plugin.go](https://github.com/Depado/go-b0tsec/tree/master/plugins/example/plugin.go) file which is fairly well documented.  

To create more complex commands (with state stored in the `Plugin` structure for example) you can take a look at the already written plugins such as `ud` (the Urban Dictionnary plugin).

To write a plugin you need to satisfy the Plugin interface which is defined like that :

```go
type Plugin interface {
	Get(*irc.Connection, string, string, []string)
	Help(*irc.Connection, string)
}
```

You will then need to register your plugin. There are two techniques to do that :
```go
RegisterCommand("myplugin", myplugin.NewPlugin())
// Or
RegisterCommand("myplugin", new(myplugin.Plugin))
```
I would advise to always use the first method as it allows you to initialize some data when registering your plugin. This is usually done in the [plugins/init.go](https://github.com/Depado/go-b0tsec/tree/master/plugins/init.go) file but you can register your plugins anywhere in your code.

## Middlewares

A middleware is a function that is executed each time a message is received by the bot.  

I see middlewares as a way to monitor and react to things that are being said on a channel without the users specifically calling for the bot. As an example you can check the github middleware that will send the description of a github repository each time it finds a github link in a message.

**Note : These middlewares are not chained. (One middleware doesn't impact the other ones)**

To write your own middleware you need to satisfy the Middleware interface which is defined like that :

```go
type Middleware interface {
	Get(*irc.Connection, string, string, string)
}
```

As for the plugins, I'd advise you to create a package per middleware, and call the middleware struct "Middleware" to keep the API stable and uniform.  
You can then register your middleware like that :

```go
RegisterMiddleware(mymiddleware.NewMiddleware())
// Or
RegisterMiddleware(new(mymiddleware.Middleware))
```

I'd also recommend the first way of registering your middleware as it allows you to initialize some data in your `Middleware` struct.

## Mixins

Sometimes you'll want to link the plugin capabilities with the middleware capabilites. One example is the `markov` plugin. It both parses everything that is sent on the channel, and also has the `!markov` command to ask for a random generated message. Keep your code clean and separate the plugin and the middleware by creating two separate files (`middleware.go` and `plugin.go`) even if they are both in the same package.

## Available Plugins and Middlewares

Each plugin and/or middleware can have its own README.md file living inside it. If there is no explanation on the plugin it means that it's simple enough to get the basics by just reading the comments.

 - [afk : Tell the world you're away. For a specific reason. Or not.](https://github.com/Depado/go-b0tsec/tree/master/plugins/afk)
 - [anon : Saying thins anonymously on the channel the bot lives in.](https://github.com/Depado/go-b0tsec/tree/master/plugins/anon)
 - [dice : Throw some dices. (Don't lie, I know you want to.)](https://github.com/Depado/go-b0tsec/tree/master/plugins/dice)
 - [ddg : Search stuff directly on duckduckgo.](https://github.com/Depado/go-b0tsec/tree/master/plugins/duckduckgo)
 - [karma : Take or give karma points to people.](https://github.com/Depado/go-b0tsec/tree/master/plugins/karma)
 - [logger : Log the whole history and links that are sent to files.](https://github.com/Depado/go-b0tsec/tree/master/plugins/logger)
 - [markov : Help the bot learn and speak to it. It really tells random stuff. ](https://github.com/Depado/go-b0tsec/tree/master/plugins/markov)
 - [seen : Check if someone is afk or the time since his/her last message.](https://github.com/Depado/go-b0tsec/tree/master/plugins/seen)
 - [title : Grab the pages of the links that are being sent on the channel.](https://github.com/Depado/go-b0tsec/blob/master/plugins/title)
 - [ud : Interrogate the Urban Dictionnary to improve your knowledge.](https://github.com/Depado/go-b0tsec/tree/master/plugins/urban)
 - [youtube : Grab relevant information on the youtube videos that are sent on the channel.](https://github.com/Depado/go-b0tsec/tree/master/plugins/youtube)
 - [github : Grab some information about repositories, pull requests, commits when someone sends a github link !](https://github.com/Depado/go-b0tsec/tree/master/plugins/github)

## Contributing

I'm open to any kind of issue and pull requests. Pull requests will only be accepted if they respect the coding style described above and if the plugin/middleware is relevant. You can also modify the core of the bot, although I'll be slightly more exigent about what you put in there. Otherwise you can just tweak the bot to fit your own needs and never make a pull request, I'm also fine with that.

## License
```
DO WHAT THE FUCK YOU WANT TO PUBLIC LICENSE
		Version 2, December 2004

Copyright (C) 2004 Sam Hocevar <sam@hocevar.net>

Everyone is permitted to copy and distribute verbatim or modified
copies of this license document, and changing it is allowed as long
as the name is changed.

DO WHAT THE FUCK YOU WANT TO PUBLIC LICENSE
TERMS AND CONDITIONS FOR COPYING, DISTRIBUTION AND MODIFICATION

0. You just DO WHAT THE FUCK YOU WANT TO.
```
