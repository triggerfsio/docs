Introduction
############

.. attention::

   triggerFS is in beta-testing right now. The pricing plan is disabled and all signups are made with the free-tier with **no limitations** and **all features enabled**. Enjoy!

Welcome to triggerFS
--------------------

triggerFS is a distributed, realtime message passing and trigger system. triggerFS enables you to build distributed systems and do realtime messaging in a service-oriented fashion. It is made up of four modules:

- worker
- cli
- client
- fs

Each one of these modules play a key role in your triggerFS environment.

Media
-----

.. raw:: html

    <div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto;">
        <iframe src="https://www.youtube.com/embed/vvfowPHD_7s" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
    </div>


Modules
#######

worker
------
A worker is or has a service and receives messages (requests) from one or more clients in a safe and concurrent way and executes them by using a plugin. Deploy a worker to a bunch of servers and connect them together, build a network of services for high-speed messaging, to distribute workload or create clusters or groups of workers.

A plugin is a tiny binary (written and compiled in go) which does one thing and does it well. A plugin could be anything:

- compute something
- filter, grep, grok, sort or manipulate data
- write to a file
- log to a file
- forward a message
- read logs
- collect metrics (eg. send metrics to graphite)
- just echo back the received message
- monitor the server the worker is running on
- execute something on a remote machine via ssh
- run a chef/puppet/fabric recipe or ansible playbook
- run a bash script
- send an SMS via twillio
- send a text message to telegram
- flip a switch (0/1 flipflops)
- start/stop/restart a service
- kill a process
- chain requests by writing to another trigger file (this is fun)
- ...

you name it. As you can see the job of a plugin is to make our life easier and serve us with mostly things that can be automated or are needed on-demand. You can write your own plugins or search for existing plugins on the marketplace (more about that below).

Every worker has an identity and a token to authenticate against the broker.

.. note::

   After successfully authenticated, a worker's identity is referred to as a ``service``. We do not distinguish between them.
   However, a service you define in the ``cli`` is a sort of reverse proxy or alias for a worker (a real service).
   The identity of a worker is really only a string acting as an identifier.

   Just remember, that once a worker is online, it's identity becomes a service. Hence the flag ``-service`` in the client module. You can pass both a real service or an identity of a worker to it.

cli
---
The cli module is an interactive cli-based management console for managing everything on your triggerFS platform. With the cli module you can:

- create new users
- create new workers
- create new services
- make services public (public services feature)
- select the message dispatching algorithm (loadbalancing and more)
- create triggers
- make requests to your services in realtime
- and much more

The triggerFS cli module is the central place for managing, orchestrating and configuring your triggerFS environment.

Suppose we wanted to do the following:
If we know that there will be a service called demo with a command plugin and three workers behind that service (announcing that service), we could create a trigger named command, attach it to the demo service, add the demo service to all of our three workers and set a timeout of - let's say - 10s. Now we have created a trigger which is represented as a file in our fs module.


client
------
A client sends messages (requests) to a service and gets back a response. The client can specify which service it wants to talk to and what plugin shall be used. The client module is simple but powerful.


fs
--
The fs module is a module for mapping the above mentioned triggers to files with the help of FUSE. It was built to go a step further than just sending messages back and forth. It enables machine-to-machine communication. Mounting files is cheap and doing socket communication by using files makes this module attractive to embedded devices or small computers like the Raspberry Pi™.

Create a directory and define a trigger in that directory in your cli. Now, if you mount the fs module to a place on your filesystem, you end up with a file in that directory within that mountpoint. Every write to that file (with the content being the data written to that file) will send a request to the workers behind the above defined service with all the predefined set of rules we configured ealier. The fs module aims to make triggerFS app-friendly in such a way that other applications can use files as their way to send a message to your services.

For example:
A trigger file could be defined in such a way that the result would be a logging of the request being sent to a service. Now we could tell Nginx to log into our trigger file instead of /var/log/nginx/*. Now everytime Nginx wants to log someting, it makes a syscall (write) to our file which would result in a message being sent. Our service would then write it to eg. a central NFS server of the company which is located on the machine where the worker is running.

Another example would be a raspberry pi which collects weather data and sends it to a central server (service) by writing into the trigger-files it mounted on its filesystem. Either scripted or syscalled.
A simple echo 'somedata 31F;10°;3.2' > /mnt/triggerfs/weatherstation/rpi/station1 is enough to send your data.

What we just did is, we triggered an action by writing to a file. Hence the name trigger.

.. note::

   You can't create regular files in your mountpoint. The ``fs`` module only supports trigger-files.
   The only allowed operations are:
   
     * ``mkdir`` to create directories
     * ``mv`` to rename trigger-files
     * ``chmod`` to set unix permissions on trigger-files

There is one more module called ``broker``. This is the broker we maintain and operate in the cloud (the service behind triggerfs.io). The client/worker communication happens to be routed via the broker.
The broker is the main coordinator for every message. It takes the request from the client and dispatches it to the services accordingly.

Security
########

Communication Flow
------------------

Beside our RESTful HTTP (JSON) API for database access, we use ZeroMQ for the communication between the client/worker and the broker.
Every authenticated request to our API is done by using a JSON Web Token (JWT).

The communication/networking between clients and workers (services) are as follows:

  * client <==> broker <==> service (worker or service)

The central broker in the cloud (we, the triggerFS team) is responsible for routing the messages back and forth.
A client cannot reach a worker without the broker and vise versa.

The broker exclusively uses the JWT of the client and/or worker if it has to make some operations on behalf of either part. This means that the JWT is also being sent when a message is sent. It is part of the message.

Since a JWT in triggerFS does not include sensitive data (only metadata) it is acceptable to send a JWT over the wire. However, in future releases we want to implement channel encryption on top of the SSL/TLS HTTP API calls, so that even the zmq channels we use to communicate are also encrypted in the future.


Database
--------

Our database is powered by postgreSQL and here is a listing of what will be stored in our db:

* users with their identity and secret (we use pgcrypto and bcrypt the password/secret before it is inserted into the db)
* workers with their identities and tokens (we use uuid_v4 for the token of a worker)
* teams with their configuration settings
* services with their configuration settings
* triggers with their configuration settings

This is everything which is stored in our db. The only sensitive data is the identity/secret of a user and identity/token of a worker and we make sure to use cryptography to secure those.

In future there will be a log table for storing the output of a plugin into the db. Which will possibly hold sensitive data. We will think about how to store those in a secure way.

Maintenances
------------

If triggerfs.io schedules maintenances and/or the broker has to be shut down, all workers/services will automatically get notified and will reconnect as soon as the broker is up and running again.

In the future we will also notify the team with a notification message to their mailboxes (future feature), which they can access within their ``cli``.


Features
########

* firewall-friendly (only outbound connections being made by workers)
* build lambda functions on your own servers
* build a network of workers and services
* make use of various plugins
* fast, reliable and service-oriented networking
* high-speed, low latency and asynchronous messaging
* realtime stdout output
* cluster-enabled services
* different message passing algorithms on services: roundrobin, serial, mirror (parallel)
* messaging via regular files with the ``fs`` module (triggers)
* the ``fs`` module is available on every device (distributed, synced FUSE filesystem)
* write your own plugin
* invite others to your team and share resources with each other
* join other teams and share resources with each other
* make services public so everybody can use them

Roadmap
#######

* marketplace api for the plugin ecosystem
* marketplace web UI
* marketplace integration into ``cli`` module
* streaming services for the ``worker`` module (long-running plugins/services)
* listening feature for the ``client`` module (for streaming services)
* service broadcasting feature
* HTTP/broker gateway for making requests via HTTP
* periodic tasks via HTTP/db
* team mailboxes in ``cli`` for notifications from broker and triggerfs.io
* log tables for storing output of plugins. (history of stdouts)
* encrypted communication (messaging) channels (no content encryption)
* and much more...


Pricing
#######

triggerFS will have a three-tier plan. Here is an overview of the pricing plan:


+-----------+-----------------------+-----------------+------------------+
|           | Free                  | Basic           | Advanced         |
+===========+=======================+=================+==================+
|           | 1 team                | 1 team          | 2 teams          |
+-----------+-----------------------+-----------------+------------------+
|           | 2 workers             | 25 workers      | 100 workers      |
+-----------+-----------------------+-----------------+------------------+
|           | 1 service             | 5 services      | 20 services      |
+-----------+-----------------------+-----------------+------------------+
|           | 3 triggers            | 26 triggers     | 51 triggers      |
+-----------+-----------------------+-----------------+------------------+
|           | 2 users/team          | 9 users/team    | 25 users/team    |
+-----------+-----------------------+-----------------+------------------+
|           | ✔ Unlimited access to marketplace                          |
+-----------+-----------------------+-----------------+------------------+
|           | ✔ Join other teams                                         |
+-----------+-----------------------+-----------------+------------------+
|           | ✘ public services     | ✔ All features enabled             |
+-----------+-----------------------+-----------------+------------------+
| **Price** | free                  | $x/month        | $x/month         |
+-----------+-----------------------+-----------------+------------------+


* worker, service and trigger limits are per team
* the free tier will always be free

.. note::

  We are in the beta-testing phase. We can't exactly tell the price, but we will update it once we know how we want to charge our customers.
  Our goal is to launch this application in a beta-testing stage so we can estimate which resources will cost us how much.
  Based on that calculation we will try to offer a fair price to our customers.

If you have been using this application for a while, we would like to hear your feedback. You can reach us at feedback@triggerfs.io. Thank you.

Target Group
############

We think that devops and system administrators will love to use triggerFS due to the way it simplifies building tools such as automation systems and communication of services.

We see DCs (data centers) in general also as a target group. For example:
A triggerfs-worker as a top-of-the-rack (tor) worker which is responsible for the systems in a rack to handle deployments, automation, triggering of jobs, etc. is one of the scenarios triggerFS can fit into.

Systemadministrators can use triggerFS for maintenance purposes or devops engineers can build whole clusters for various deployment scenarios.

In the end, it will be the massive amount of plugins which will enable triggerFS to become something useful for any possibly imaginable task.

Of course everybody is welcome to try out triggerFS (there is a free-tier subscription. Go try it out!)
