Introduction
------------

You can do a lot with triggerFS. That's why we want to give you some examples and use cases. Hopefully you will find enough information here to use triggerFS the way it fits your needs.

We do have a screencast where we show a complete set up of two workers with one service in front of it and task execution with the help of the `command` core plugin.


Media
-----

.. raw:: html

    <div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto;">
        <iframe src="https://www.youtube.com/embed/vvfowPHD_7s" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
    </div>


Use Cases
#########

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
You provide your Twilio api key and api secret with the ``-args`` flag of the client module and a string. The call returns with either a success or an error and your application code continues accordingly.

Scenario 4 - A call back service
--------------------------------

You have set up asterisk as your PBX server. You have set up a service which uses a plugin that registers users by their phone numbers and whenever somebody requests your service you call them back with your PBX server.

Scenario 5 - Chaining requests
------------------------------

You want to relay messages from one server to another. You set up a service which uses a plugin that takes a request and writes it directly to a trigger-file mounted on the server where the worker is running.
That trigger-file is defined in such a way that every write to that file will trigger a request to another service, and so on. Now whenever someone sends a message to server A it ends up being received on server B.


The list goes on and on. The opportunities to use triggerFS are countless. Feel free to come up with your own ideas on how to use triggerFS.
And also feel free to tell us your story on how you use triggerFS in your environment. Drop us an email at feedback@triggerfs.io.


Examples
########

- Mass System Upgrades
  Given we have 100 servers with Ubuntu/Debian, we deploy one worker on each of them and bind them to multiple services (eg. divided by location/rack/dc). We use the `command` plugin to upgrade the systems:

  .. code:: bash

     ./triggerfs-client -service dc1 -timeout 1h -plugin command -command 'apt-get update && apt-get upgrade'


- Telegram Bot
  Given we have a telegram group with a bot inside, we deploy one worker on a server/vps and mount our trigger files on the same server with the ``triggerfs`` module.
  We write a telegram plugin to listen on that group and whenever a message is sent to the bot we write into a trigger-file which would result in sending a message to a certain service.
  We send back a message to the group that the file has been written.


- Suppose Twilio would be using triggerFS. Given that the Twilio team offers a public service to registered users for sending SMS: We have an account at Twilio and want to send an SMS to a friend.

  We want to use this public service and invoke the following command:

  .. code:: bash

     ./triggerfs-client -service twilio/api -plugin sms -command 'Hey buddy, can we meet at 3pm today?' -args recipient=0123456789 -args apikey=$MYAPIKEY_BASH_ENV -args apisecret=$MYAPISECRET_BASH_ENV


  Note the service syntax for public services here. We specify the team name and the service name separated by a slash.