Home For Plugins
################

triggerFS gives you the platform for high-speed realtime messaging. But that is only a small fraction of what this is all about, really. What really powers triggerFS is the gigantic amount of plugins you can use to do any kind of task. These plugins or the combination of several of them is what makes triggerFS a pleasure to work with.

Those plugins need a place where they can be listed to the public, so everyone can benefit and download and use them. That is our vision of building the triggerFS marketplace. Anyone who can write go (https://golang.org/) can also write plugins. Check out our core plugins so you can get a feeling of how a plugin works. Our plugin skeleton has 57 lines of code (loc). Add your own function to it and you have just created your first own plugin.

Plugins are small pieces of code, doing one thing and doing it well. We want to build as many plugins as possible for any kind of job. You can host them on github and link your repository in the triggerFS marketplace, making it accessible within the triggerFS app. Our goal is to provide you with tons of plugins with the help of the community in the near future.

The marketplace is in development right now and will be launched soon.


How The Marketplace Works
#########################

It is really simple. If you have written a plugin and you think it's worth to be used by everyone you can share it with others on the triggerFS marketplace:

* upload your plugin to github
* provide a meta json file and a README.md
* go to the marketplace and add a new plugin
* provide the URL to your github repository of that plugin
* we will fetch the informations and metadata to your repository
* we will display your plugin on the marketplace
* we will test (a real test) your plugin by building it first and then run an internal backend worker with that plugin
* we will then make a request to that worker with the "invocation" example you give in the meta json
* if everything is ok we will build a statically linked binary (no dependencies needed, such as zmq) and offer a download of the binary
* people still can choose to use the source code and build it by themselves

Community
#########

We hope that any time soon the community will offer more and more plugins, because hopefully everyone will write plugins and will love to share them.
We want to inspire the community with own work of the core plugins. Whenever we have time to build a plugin, we write and test it so we can put them into the core plugins repository.


A Free Marketplace
##################

The marketplace is mainly free to use. At least this is what we think the marketplace should look like. However, we think it can be beneficial for companies to offer their 
own plugins (think of proprietary plugins) for money. Or other users who spent a big amount of time on a plugin and want to monetize their idea/work. That's why we are 
thinking about giving the opportunity to people to place their plugin and charge users. How to charge and when is still in theory. For now we don't spend too much time 
on this idea. We can't say for sure if and when this idea will be introduced to the triggerFS marketplace.