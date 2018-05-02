=======================
Configuration & Modules
=======================

We will start with the cli module since we have nothing configured to run the other modules. Everything configuration happens in the cli.

We will first start by creating a worker, fire up the worker, create a service and a trigger.

.. note::

   The ``cli`` module's syntax is similar to Juniper's cli. The syntax and commands available in the cli are pretty much self-explanatory and its help menu will have information on what each command is doing. You can access the help menu by invoking ``help`` in both ``run`` and ``configure`` modes.
   
   The cli also has autocompletion of words. So use <TAB><TAB> (like you would do in your shell) to type fast and complete worker identities, service names, etc.

In this configuration example we assume that we are logged in as the user "hp" in the team "team1". The prompt of the cli will indicate this.

Workers
#######

First we will start with creating a worker so we can run it on a server.

Create a new worker
-------------------

.. code-block:: bash

   (team1) hp@example.com$ configure# set workers identity demo01
   (team1) hp@example.com$ configure#  
   (team1) hp@example.com$ configure# show workers
     id | identity |      description       |    state     |                token                 |   owner   |            created            |            updated             
   +----+----------+------------------------+--------------+--------------------------------------+-----------+-------------------------------+-------------------------------+
      6 | demo01   |                        | DISCONNECTED | 9b1bd278-e645-4eec-bffb-0587ad84534e | hp        | Mon, 30 Apr 2018 18:45:30 UTC | Mon, 30 Apr 2018 18:45:30 UTC  
   +----+----------+------------------------+--------------+--------------------------------------+-----------+-------------------------------+-------------------------------+
   (1 rows)

   Time: 545.092072ms
   (team1) hp@example.com$ configure#  

Update configuration file
-------------------------

Now we take the identity and token of the newly created worker and put it into our toml configuration file.

The naming of the subsection is simple: workers.$team.$identity. Then specify the token under that subsection with a key named "token".

.. code-block:: bash

   ...
   [workers]
   # path to plugins folder
   pluginspath = "/home/hp/gocode/src/github.com/triggerfsio/plugins/plugins"
   
   # subsections of [workers] section identified by identities prefixed with "workers."
   [workers.team1.demo01]
   token = "9b1bd278-e645-4eec-bffb-0587ad84534e"
   ...

.. hint::

   Workers only need the [workers] section in the config file. No need for credentials in the [main] section

Services
########

Until now the worker is already available under the service name "demo01". Remember that a worker identity automatically becomes a service in triggerFS.

Since we want to make use of a trigger file, we need to go a step further and create a real service so we can attach our worker to it.

.. note::

   trigger-files only work with real services. Workers cannot be attached to a trigger file.

Create a new service
--------------------

.. code-block:: bash

   (team1) hp@example.com$ configure# set services name demoservice
   (team1) hp@example.com$ configure# show services
     id |    name     | description | timeout | algorithm  | triggers | workers | visibility | owner |            created            |            updated             
   +----+-------------+-------------+---------+------------+----------+---------+------------+-------+-------------------------------+-------------------------------+
      2 | demoservice |             |         | roundrobin |          |         | local      | me    | Mon, 30 Apr 2018 18:49:02 UTC | Mon, 30 Apr 2018 18:49:02 UTC  
   +----+-------------+-------------+---------+------------+----------+---------+------------+-------+-------------------------------+-------------------------------+
   (1 rows)

   Time: 47.334023ms

The service is now available only to you (visibility is local by default).

Attach workers
--------------

.. code-block:: bash

   (team1) hp@example.com$ configure# set services demoservice workers add demo01
   Notice: this service has algorithm roundrobin. Roundrobin is handled by the broker. Any newly added worker to this service should reannounce its services.
   (team1) hp@example.com$ configure# show services
     id |    name     | description | timeout | algorithm  | triggers | workers | visibility | owner |            created            |            updated             
   +----+-------------+-------------+---------+------------+----------+---------+------------+-------+-------------------------------+-------------------------------+
      2 | demoservice |             |         | roundrobin |          | demo01  | local      | me    | Mon, 30 Apr 2018 18:49:02 UTC | Mon, 30 Apr 2018 18:49:02 UTC  
   +----+-------------+-------------+---------+------------+----------+---------+------------+-------+-------------------------------+-------------------------------+
   (1 rows)

   Time: 51.229509ms
   (team1) hp@example.com$ configure#  


.. note::

   Notice how the cli is telling us to let any newly added workers to reannounce their services. Since the worker we have just created never ran, it will automatically announce ``demoservice`` on startup.

Start the worker
----------------

Let's start the worker on any server. Remember to deploy the toml configuration file to the server.

.. code-block:: bash

   hp@localpc $ ./triggerfs-worker -identity demo01 -debug true
   2018/04/30 21:03:01 I: connecting to broker at tcp://triggerfs.io:5555...
   2018/04/30 21:03:01 I: trying to connect recv socket to broker
   2018/04/30 21:03:01 I: trying to connect send socket to broker
   2018/04/30 21:03:02 I: received BROKER_ACCEPTED from broker for worker socket - Connected
   2018/04/30 21:03:02 I: Found service demoservice. Announcing it to broker.
   2018/04/30 21:03:05 I: received HEARTBEAT for worker socket from broker.
   2018/04/30 21:03:07 I: received HEARTBEAT for worker socket from broker.
   2018/04/30 21:03:10 I: received HEARTBEAT for worker socket from broker.

Notice how the worker has recognized that it is attached to the service ``demoservice`` and announced it to the broker. Ready to listen on it.
A quick look in the cli tells us that the worker is online:

.. code-block:: bash

   (team1) hp@example.com$ configure
   (team1) hp@example.com$ configure# show workers
     id | identity |      description       |    state     |                token                 |   owner   |            created            |            updated             
   +----+----------+------------------------+--------------+--------------------------------------+-----------+-------------------------------+-------------------------------+
      6 | demo01   |                        | ONLINE       | 9b1bd278-e645-4eec-bffb-0587ad84534e | hp        | Mon, 30 Apr 2018 18:45:30 UTC | Mon, 30 Apr 2018 19:03:01 UTC  
   +----+----------+------------------------+--------------+--------------------------------------+-----------+-------------------------------+-------------------------------+
   (4 rows)

   Time: 33.375014ms
   (team1) hp@example.com$ configure#  

.. attention::

   **Important**: Remember to have the command plugin installed on the server where the worker is running. Have a look at Plugins_ for more information.


Trigger
#######

Create new directory
--------------------

All trigger-files must be within a directory under root (/). So first, create a directory if you haven't already:

.. code-block:: bash

   (team1) hp@example.com$ configure# ^D
   (team1) hp@example.com$ file
   >> Starting a new interactive shell
   hp@localpc /tmp/triggerfs-client403096611  $ mkdir newtrigger
   hp@localpc /tmp/triggerfs-client403096611  $ <CTRL+D>
   (team1) hp@example.com$  


Create new trigger
------------------

.. code-block:: bash

   (team1) hp@example.com$ configure
   (team1) hp@example.com$ configure# set trigger name /newtrigger/demotrigger
   (team1) hp@example.com$ configure#  


Configure trigger
-----------------

Since a trigger is just a set of definitions to what shall happen if the trigger-file is being written, we need to define them first:

.. code-block:: bash

   (team1) hp@example.com$ configure# set trigger demotrigger plugin command/command
   (team1) hp@example.com$ configure# set trigger demotrigger service attach demoservice
   (team1) hp@example.com$ configure#  
   
Now we've defined that this trigger shall route the messages to the service called ``demoservice`` (where the worker demo01 sits behind and listens) and that the plugin command/command should be used.
Note that ``command/command`` is the actual path to the directory where the plugin (binary) is located. Since the pluginspath in the configuration file is configured as ``/home/hp/gocode/src/github.com/triggerfsio/plugins/plugins`` it looks for a binary in ``/home/hp/gocode/src/github.com/triggerfsio/plugins/plugins/command/`` named ``command``.

.. code-block:: bash

   (team1) hp@example.com$ configure# show triggers
      id |    name     | description |     plugin      | hits |   owner   | visibility |            created            |            updated             
    +----+-------------+-------------+-----------------+------+-----------+------------+-------------------------------+-------------------------------+
       2 | demotrigger |             | command/command |    0 | hp        | local      | Mon, 30 Apr 2018 18:53:54 UTC | Mon, 30 Apr 2018 18:53:54 UTC  
    +----+-------------+-------------+-----------------+------+-----------+------------+-------------------------------+-------------------------------+
   (1 rows)

   Time: 207.095606ms
   (team1) hp@example.com$ configure#  

If we go back into our filesystem where the fs module has mounted our triggerFS filesystem, we will see that a new file is located under the ``newtrigger`` directory:

.. code-block:: bash

   (team1) hp@example.com$ configure# ^D
   (team1) hp@example.com$ file
   >> Starting a new interactive shell
   hp@localpc /tmp/triggerfs-client403096611  $ ll newtrigger/
   total 512
   -rw-r--r-- 1 hp hp 0 Apr 30 20:53 demotrigger
   hp@localpc /tmp/triggerfs-client403096611  $ <CTRL+D>
   (team1) hp@example.com$  


Client
######

Make a request
--------------

Let's make a request to our new service with the client module. We will define our service, the plugin to be used and a timeout for the request. Our command will be ``uptime`` to get the uptime of the server:

.. code-block:: bash

   hp@localpc $ ./triggerfs-client -service demoservice -plugin command/command -timeout 10s -command uptime
   2018/04/30 21:14:06 Sending message to service demoservice (roundrobin)
   [command/command@demo01]  21:14:07 up  8:26,  7 users,  load average: 0.61, 0.60, 0.52
   
   Exit code: 0
   Total messages: 1
   Time ran: 907.914622ms
   
   hp@localpc $ 

The response came from the server with the worker running on called ``demo01`` and the plugin ``command/command`` and the output of the ``uptime`` command.

.. hint::

   The client module also reads stdin, so you can skip the ``-command`` flag and echo uptime piped to the client:

.. code-block:: bash

   hp@localpc $ echo uptime | ./triggerfs-client -service demoservice -plugin command/command -timeout 10s
   2018/04/30 21:14:06 Sending message to service demoservice (roundrobin)
   [command/command@demo01]  21:14:07 up  8:26,  7 users,  load average: 0.61, 0.60, 0.52
   
   Exit code: 0
   Total messages: 1
   Time ran: 907.914622ms
   
   hp@localpc $ 


FS
##

Now, since we have set up a trigger we can also use the fs module to write to a real file.

Run
---

First we start the module which will always run in the foreground (there is no background mode currently):

.. code-block:: bash

   hp@localpc $ ./triggerfs
   triggerfs (v1.0.0)
   
   **********************************************************************************
   *** Welcome to triggerFS. A realtime messaging and distributed trigger system. ***
   **********************************************************************************
   
   2018/04/30 21:20:33 === triggerfs module started ===
   2018/04/30 21:20:33 No JWT provided. Authenticating with login credentials in config.
   2018/04/30 21:20:34 Successful login. JWT is eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
   2018/04/30 21:20:34 Successful login.
   
   2018/04/30 21:20:34 Started triggerfs.
   2018/04/30 21:20:34 Serving filesystem in ./mountpoint
   2018/04/30 21:20:34 Log file is ./triggerfs.log
   2018/04/30 21:20:34 Ready and running in foreground...

The mountpoint in this case is the directory called ``mountpoint`` in $PWD (set in the configuration toml file as ./mountpoint).


Execute (write to trigger-file)
-------------------------------

Now in another terminal we can go into that directory and write to the trigger-file:

.. code-block:: bash

   hp@localpc $ ll mountpoint/
   total 512
   drwxrwxr-x 1 hp hp 0 Apr 30 20:52 newtrigger/
   hp@localpc $ ll mountpoint/newtrigger/
   total 512
   -rw-r--r-- 1 hp hp 0 Apr 30 20:53 demotrigger
   hp@localpc $ echo uptime > mountpoint/newtrigger/demotrigger 
   hp@localpc $ 

Since we cannot write into stdout in FUSE (except we have read from a file) the output (response) of this request will be displayed in the terminal where the fs module is running in foreground.

If a logfile was specified in the configuration file for the ``[triggerfs]`` section then the response can be found there as well.

A look at the trigger in our cli will show that it got one hit:

.. code-block:: bash

   (team1) hp@example.com$ configure# show triggers
     id |    name     | description |     plugin      | hits |   owner   | visibility |            created            |            updated             
   +----+-------------+-------------+-----------------+------+-----------+------------+-------------------------------+-------------------------------+
      2 | demotrigger |             | command/command |    1 | hp        | local      | Mon, 30 Apr 2018 18:53:54 UTC | Mon, 30 Apr 2018 19:22:52 UTC  
   (1 rows)
   
   Time: 177.43962ms
   (team1) hp@example.com$ configure# 

in the ``hits`` column.


This was one configuration flow in its simplest form for a complete setup of a trigger.

We have created a worker, bound it to a service, created a trigger with a specified set of rules and executed a request in both ways with the client and the fs module.



Plugins
#######

Plugins are binaries written and compiled in go. Since triggerFS uses zmq for its socket communication, you need to install libzmq3.

This is only necessary if you want to build the plugin yourself. The triggerFS marketplace later will have binaries pre-built (statically linked, so you don't need any dependencies) for you (beside the available source code of the plugin).

The triggerFS core plugins are available at https://github.com/triggerfsio/plugins.

Go get them with ``go get``:

.. code-block:: bash

   go get github.com/triggerfsio/plugins

.. note::

   The core plugins currently come without a pre-built binary. We will save this feature for later when we have launched the marketplace.

   For now, you **have to** build the core plugins yourself. If you are experienced with docker, you can also make use of the golang docker file which comes with golang already installed.
   In this example we will assume that you have installed golang and libzmq3-dev on your machine.

Dependencies
------------

If you want to build a plugin you need to have installed libzmq3 and its developer files. Install it with:

.. code-block:: bash

   apt-get install libzmq3-dev

Build
-----

Now switch to the folder of the plugin you want to build:

.. code-block:: bash

   hp@localpc ~ $ cd gocode/src/github.com/triggerfsio/plugins/
   hp@localpc ~/gocode/src/github.com/triggerfsio/plugins $ cd plugins/command/
   hp@localpc ~/gocode/src/github.com/triggerfsio/plugins/plugins/command $ go build command.go 
   hp@localpc ~/gocode/src/github.com/triggerfsio/plugins/plugins/command $ ll
   total 8.2M
   -rwxrwxr-x 1 hasan hasan 8.2M Apr 30 21:33 command*
   -rw-rw-r-- 1 hasan hasan 2.3K Apr  4 00:58 command.go
   hp@localpc ~/gocode/src/github.com/triggerfsio/plugins/plugins/command $ 

Now you can point your plugins folder to this directory (in your toml configuration file under section ``[workers]``):

.. code-block:: bash

   ### WORKERS SECTION
   [workers]
   # path to plugins folder
   pluginspath = "/home/hp/gocode/src/github.com/triggerfsio/plugins/plugins"
   
   # subsections of [workers] section identified by identities prefixed with "workers."
   [workers.team1.demo01]
   token = '9b1bd278-e645-4eec-bffb-0587ad84534e'
   ...

The command binary will be ready for use now.