Core Plugins
############

Please refer to `Configuration#Plugins <configuration.html#plugins>`_ to see how to get and configure the triggerFS core plugins.


Custom Plugins
##############

In this section you will learn how to write your own plugins and build them so your workers can use them.

Plugins are written in go. You will need golang (https://golang.org) to be able to write plugins.

Writing plugins should be as easy as possible. That's why we offer you a skeleton that you can copy and paste into your editor and start writing your plugin from there.

Let's start with getting copying the skeleton plugin code available in the official triggerFS plugins repository at https://github.com/triggerfsio/plugins.

If you haven't installed go please do so first.


1. Go get the repository via ``go get``:

.. code-block:: bash

   go get github.com/triggerfsio/plugins


2. Copy the skeleton plugin code to a new project (directory) and open it in your favorite editor:

.. code-block:: bash

   cd $GOPATH/src/github.com/triggerfsio/plugins
   cp skeleton/plugin.go ~/mynewplugin/main.go

3. Write your own function in the ``Init()`` method of the plugin.

4. Build your plugin with go. Optionally give it an output name ``myplugin``:

.. code-block:: bash

   cd ~/mynewplugin
   go build main.go -o myplugin

5. Set your pluginspath in the configuration toml file:

.. code-block:: bash

   pluginspath = "/home/hp/mynewplugin"

**Note**: you must use absolute paths in your toml file. $HOME/mynewplugin would not work here.

6. Make a request to the service and specify your own plugin:

.. code-block:: bash

   ./triggerfs-client -service myworker_or_service -plugin myplugin -timeout 3s
