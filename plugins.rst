Core Plugins
############

The core plugins are the official plugins written and maintained by the triggerFS team. They are well tested and known to be bug-free and working.

We will include more and more plugins into the core plugins repository, so we can have a nice set of plugins to work with.

We hope that the rest will be done by the community with their great ideas and great plugins shared to the public.

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

Let's say we want to write the payload to a file and return the message "file has been written." back to the client.
We also want to notify the client in realtime that we are about to write to that file.

.. code-block:: go

        // Init implements the triggerfs plugin Interface
        func (ap *AwesomePlugin) Init(message *plugins.Message, resp *plugins.Response) error {
                // open a channel for realtime communication back to the client.
                err := ap.Plugin.Open(message.Socket)
                if err != nil {
                        return err
                }
                // IMPORTANT: remember to defer close the channel or you will get timeouts!
                defer ap.Plugin.Close()

                data := message.Command[0]
                filepath := message.Args["filepath"]

                // now implement your plugin here
                ap.Plugin.Send(fmt.Sprintf("Writing payload: %s to file %s now", data, filepath))
                err = ioutil.WriteFile(filepath, []byte(data+"\n"), 0644)
                if err != nil {
                        log.Printf("failed to write to file: %s\n", err)
                }

                // and finally set the exitcode and a final message on resp
                resp.ExitCode = 0
                resp.Output = []string{"file has been written."}

                return nil
        }

We have defined data and filepath here. data is the command being sent by the client with the ``-command`` flag or stdin and filepath is the plugin argument specified by the client.
We then notify the client that we will write now. Then we do the actual writing to the file. And finally we return from the function after we set our response output (that the file has been written).

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

   $ ./triggerfs-client -service hp01 -plugin myplugin -timeout 3s -command "hello world" -args filepath=/tmp/myfile.txt
   2018/05/02 00:52:48 Sending message to service hp01
   [myplugin@hp01] Writing payload: hello world to file /tmp/myfile.txt now
   [myplugin@hp01] file has been written.
   
   Exit code: 0
   Total messages: 2
   Time ran: 118.079784ms
   
   $ 

If we cat the file /tmp/myfile.txt we see the following:

.. code-block:: bash

   $ cat /tmp/myfile.txt
   hello world
   $


Congratulations! You have just written your first triggerFS plugin. Of course this one was really simple. A plugin can vary from simple to super-complex stuff.
That's why plugins enable you to do so many things.