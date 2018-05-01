Introduction
############

triggerFS is a distributed, realtime message passing and trigger system. triggerFS enables you to build distributed systems and do realtime messaging in a service-oriented fashion. It is made up of four modules:

- worker
- cli
- client
- fs

Each one of these modules play a key role in your triggerFS environment.

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

Pricing
#######

triggerFS will have a three-tier plan. Here is an overview of the pricing plan:


+----------+--------------+----------------+----------------+
|          | Free         | Basic          | Advanced       |
+==========+==============+================+================+
|          | - 1 team     | - 1 team       | - 2 teams      |
+----------+--------------+----------------+----------------+
|          | - 2 workers  | - 24 workers   | - 50 workers   |
+----------+--------------+----------------+----------------+
|          | - 1 service  | - 12 services  | - 25 services  |
+----------+--------------+----------------+----------------+
|          | - 5 triggers | - 25 triggers  | - 50 triggers  |
+----------+--------------+----------------+----------------+
| Features |              | - 5 triggers   | - 25 triggers  |
+----------+--------------+----------------+----------------+
|          | - Unlimited access to marketplace              |
+----------+--------------+----------------+----------------+
| Price    | Free         | N/A            | N/A            |
+----------+--------------+----------------+----------------+

The pricing plan is not finished, yet. So we can't exactly tell the price, but we will update it once we know how we want to charge our customers.

**Attention**: We want to start with the free tier with all features enabled so we can offer you the full featured experience of triggerFS.

Target Group
############

We think that devops and system administrators will love to use triggerFS due to the way it simplifies building tools such as automation systems and communication of services. We see DCs (data centers) in general also as a target group. A triggerfs-worker as a top-of-the-rack (tor) worker which is responsible for the systems in a rack to handle deployments, automation, triggering of jobs, etc. is one of the scenarios triggerFS can fit into. Of course everybody is welcome to try out triggerFS (there is a free-tier subscription. Go try it out!)
