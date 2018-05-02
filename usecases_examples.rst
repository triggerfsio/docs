Why triggerFS?
##############

TriggerFS is here to help you with the right tool. We don't want to invent anything new, we want to take already existing parts and put them together to build a system which can deliver the experience many people are looking for when:

- there is a new problem to be solved
- building distributed systems can actually be tricky
- suddenly automating things becomes relevant
- communication across boundaries is necessary but firewalls are in your way
- tasks need to be outsourced to other systems
- tasks need to be scheduled
- periodic tasks need to be run
- geo-replication is a topic
- a "serverless" design is taken into consideration
- lambda functions on own servers are needed
- fire&forget-style events could be needed
- wanting a central place where everything can be run (triggered) from

Benefits
--------

triggerFS offers you one more way to access your server or services on that server.
If ssh is the door to get into your server, think about triggerFS as a door to access various, selected services on that server.

triggerFS' ``fs`` module also helps to enable M2M (machine-to-machine) communication on embedded devices. Since writing to files is a cheap operation, 
embedded devices with limited resources can do socket communication easily with triggerFS.

By inviting other triggerFS users into your team you can collaborate with each other and share resources within the team.
For example: Different universities can come together and share their servers to do data crunching or data analysis.

By joining other teams you can share your services team-wide with other users of that team. This way you can make your service accessable only for a set of users.

By making your service public you can offer your service to the whole world. People can then access your services either with the triggerFS ``client`` module or 
the HTTP/broker gateway via HTTP to make requests to your service.


Examples
########

Scenario 1 - Logging
--------------------

You have installed and set up Graylog as a log-aggregator and want to log to it with the GELF format whenever somebody hits your nginx webserver.
You define a path in your nginx configuration to log to a trigger-file in your mountpoint anywhere on your server where the webserver is runing.

You use a plugin which takes data, packs it into a GELF format and sends it to the graylog server. Now whenever nginx logs a hit to your website, a message with the log content will be logged to graylog.

Scenario 2 - Embedded devices
-----------------------------

You have a Raspberry Pi running somewhere outdoors which measures the outside temperature. You define a trigger which sends a message to a specific service with the appropriate plugin.
The plugin takes the temperature and stores it into your MySQL database. Now everytime your Raspberry Pi measures the temperature it writes to a trigger-file mounted on its filesystem and the temperature ends up in the MySQL database.

Scenario 3 - Lambda functions
-----------------------------

You want to use a lambda function in a serverless way. Your application code reaches a point where it has to send an SMS to a user. You make use of the client or the HTTP/broker gateway to send a request to your service and use a plugin which uses twilio as an SMS provider.
You provide your twilio api key and api secret with the ``-args`` flag of the client module and a string. The call returns with either a success or an error and your application code continues accordingly.

Scenario 4 - A call back service
--------------------------------

You have set up asterisk as your PBX server. You have set up a service which uses a plugin that registers users by their phone numbers and whenever somebody requests your service you call them back with your PBX server.

Scenario 5 - Chaining requests
------------------------------

You want to relay messages from one server to another. You set up a service which uses a plugin that takes a request and writes it directly to a trigger-file mounted on the server where the worker is running.
That trigger-file is defined in such a way that every write to that file will trigger a request to another service, and so on. Now whenever someone sends a message to server A it ends up being received on server B.


The list goes on and on. The opportunities to use triggerFS are countless. Feel free to come up with your own ideas on how to use triggerFS.
And also feel free to tell us your story on how you use triggerFS in your environment. Drop us an email at feedback@triggerfs.io.
