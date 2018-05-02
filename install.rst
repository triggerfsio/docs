Pre-requirements
################

Despite of triggerFS being a SaaS application. It comes without a website or SPA (Single Page Application) like most SaaS do.
Instead, the whole application is based on cli. So it was only right to continue with this philosophy and implement the Sign-Up process
into a cli installer. That is why triggerFS comes with a cli based installer. 

The cli installer makes use of dialog. So make sure you have dialog installed if it does not come with your distribution by default. Also you need wget and curl.

.. code-block:: bash

   apt-get install dialog wget curl

to install dialog on debian-based distributions.

The installer will notify you anyway if at least one of the above packages is missing.

The Installer
#############

One-liner
---------

The installer can be started with a one-liner directly from the triggerfs.io server:

.. code-block:: bash

   curl https://install.triggerfs.io | bash

The script behind https://install.triggerfs.io is the same as the ``ìnstall.sh`` script in the official triggerFS packages repository at https://github.com/triggerfsio/packages.

Manual
------
Download install.sh from the triggerFS packages repository or clone the whole repository and run the ``install.sh`` script:

.. code-block:: bash

   git clone https://github.com/triggerfsio/packages.git
   cd packages
   ./install.sh


Signup
------
Signing up is really simple and takes less than a minute. Just choose "Signup" from the menu and follow the instructions of the installer.
You will be notified with an email if you have successfully signed up.

Team creation
-------------
You can choose to create a team after you have successfully signed up and activated your account with the activation code which was sent to you via mail.
You can also create a team in your cli module. See `Configuration <configuration.html>`_ for more information on how to use the cli.


Download modules
-----------------
To get all binaries (modules: worker, cli, client and fs) choose "Download modules" and specify a directory which must exist on your filesystem.
The binaries come as statically linked. This means you don't need any dependencies on your system to be able to run the modules.

However, you need *libfuse2* for the modules ``cli`` and ``fs`` since both of them use FUSE to mount trigger-files onto your filesystem. Install it:

.. code-block:: bash

   apt-get install libfuse2


Configuration File
##################
Important thing to know is that all triggerFS modules use the same configuration file for its configuration directives. The configuration file is a toml file called ``triggerfs.toml`` and is searched in $HOME by default.

Get the skeleton configuration toml file from https://github.com/triggerfsio/packages:

.. code-block:: bash

   wget https://raw.githubusercontent.com/triggerfsio/packages/master/triggerfs.toml

or copy triggerfs.toml from the git repository we previously cloned:

.. code-block:: bash

   cd packages
   cp triggerfs.toml ~


Edit configuration file
-----------------------

Replace your credentials in the main section of the configuration file:

.. code-block:: bash

   ### MAIN SECTION
   [main]
   # team name for login
   team = "myawesometeam"
   # identity (email address) for login
   identity = "demo@example.com"
   # password for login
   secret = "secret"
   ### MAIN SECTION END

We will mention the configuration file a few more times in the Configuration section of this documentation.