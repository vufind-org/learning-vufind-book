Part 2. Configuration and Administration
****************************************

Chapter 6. Administering a VuFind Server
########################################

Learning Objectives
---------------------------------------

After reading this chapter, you will understand:

•  How to ensure that necessary processes start up automatically after your server reboots
•  How to manage VuFind’s Solr index updates.
•  Which maintenance tasks to set up for smooth operation of a VuFind instance

6.1 Introduction
________________

It is fairly easy to get a VuFind instance up and running, but there are some potential problems that only become apparent with time. How do you ensure that Solr starts automatically each time you reboot your server? How do you manage rebuilds of your index when you want to change something? How do you keep your index up to date automatically? How do you ensure that VuFind cleans up after itself properly and doesn’t fill up your disk? This chapter answers these and other questions relating to running a production VuFind server for the long term.

6.2 Starting Solr Automatically
-------------------------------

As discussed in 2.2.2, VuFind relies on a running Solr index to allow users to search locally-stored information. While it is simple to manually start the Solr process for testing and development purposes, once you deploy VuFind to production, you will want to ensure that Solr is always running so that users do not experience an interruption to service. An important part of this is ensuring that Solr always starts running when your server boots up, so the service is always available after planned or unplanned server restarts.

This chapter describes how to set up Solr to start automatically using systemd, the startup system found in the latest releases of most Linux distributions as of this writing. The process is significantly different in Windows and on older Linux distributions; for help with those, see the Starting and Stopping Solr page in the VuFind wiki (https://vufind.org/wiki/administration:starting_and_stopping_solr).

6.2.1 Creating a Solr User
__________________________

The first thing you will need to do is to decide on a Linux user account that will be used to run Solr. For security reasons, it is best to run Solr using its own dedicated user account; this way, you can limit which parts of the file system Solr is allowed to read and write, which will limit the amount of damage that can be done if the security of your Solr process is ever compromised.

To set this up, you should first create a user with the adduser command. For the purposes of this chapter, we will call the user “solr,” but you can use whatever name you like.

.. code-block:: console

   sudo adduser solr


Once the user account exists, you should change ownership of VuFind’s Solr directory to the new user so that Solr will have appropriate read/write access to its index files.

.. code-block:: console

   sudo chown -R solr:solr $VUFIND_HOME/solr/vufind


6.2.2 Creating the Service File
_______________________________

Programs that can be started or stopped using systemd are known as services. Each is defined by a small configuration file found in /etc/systemd/system which tells the operating system how to manage the program. To manage VuFind’s Solr instance, you can create a file called /etc/systemd/system/vufind.service, containing the following:

.. code-block:: console

   After=network.target

   [Service]
   Type=forking
   ExecStart=/bin/sh -l -c '/usr/local/vufind/solr.sh start' -x
   PIDFile=/usr/local/vufind/solr/vendor/bin/solr-8983.pid
   User=solr
   ExecStop=/bin/sh -l -c "/usr/local/vufind/solr.sh stop" -x
   SuccessExitStatus=0
   LimitNOFILE=65000
   LimitNPROC=65000

   [Install]
   WantedBy=multi-user.target

Be sure that the file paths in ExecStart and ExecStop point to the correct command in $VUFIND_HOME. Confirm that the User line matches the user account you created in 6.2.1 above. For an explanation of the remaining settings, see the systemd.service documentation (https://www.freedesktop.org/software/systemd/man/systemd.service.html).

6.2.3 Starting, Stopping and Enabling the Service
_________________________________________________

Once you have created your vufind.service file in the appropriate place, you can manage the service using the systemctl command. To start Solr, you can now run:

.. code-block:: console

   sudo systemctl start solr

Similarly, to shut down the service, you can run:

.. code-block:: console

   sudo systemctl stop solr

Finally, to enable the service so that it always starts when your server reboots, you can run:

.. code-block:: console

   sudo systemctl enable solr

6.3 Rebuilding/Resetting the Solr Index
---------------------------------------

There are a variety of reasons that you may eventually want to rebuild your Solr index. When upgrading to a new version of VuFind, it will sometimes be necessary to reindex to reflect changes to VuFind’s Solr schema or updates to the included version of Solr. You may accidentally load bad data into the index and need to create a fresh copy. After months or years of automated synchronization (see 6.4 below), your index may get out of sync with the system that you use to manage your records, and you may wish to rebuild to be sure everything is accurate and up to date. Whatever the reason for rebuilding the index, this section will show you how to do it safely and easily.

6.3.1 Resetting the Solr Index
______________________________

If you simply want to empty out your Solr index and start over, this is very simple. Each Solr core stores data in a directory called “index,” possibly supplemented by one or more spell-check directories with names beginning with “spell.” Resetting a core is a three-step process:


1.      Stop the Solr service
2.      Delete the index and spell-check directories
3.      Start the Solr service

When you restart Solr after deleting its index files, it will automatically initialize a new empty index for you. So, for example, if you wanted to reset your biblio core, you could run these commands:

.. code-block:: console

   systemctl stop solr
   sudo rm -rf $VUFIND_HOME/solr/vufind/biblio/index 
   sudo rm -rf $VUFIND_HOME/solr/vufind/biblio/spell*
   systemctl start solr

This will leave you with a fresh, empty index, ready for records to be indexed into it.

6.3.2 Rebuilding a Solr Index with Minimal Service Interruption
_______________________________________________________________

Indexing large collections can take a significant amount of time. If you are running a production system, you do not want to cut off your users’ access to search capabilities for long periods of time just because you need to rebuild your index. Fortunately, if you have access to another system, you can take advantage of the way Solr stores its index to rebuild your index with a minimum of service disruption.

Solr’s index is stored as files on disk, and these files are “portable” – all you have to do to copy a Solr index from one server to another is to copy the core directory containing the index.

When you run a service in production, it is a good practice to maintain a “staging” server that you can use for testing upgrades and customizations before you deploy them to your users. Having a staging server can also be valuable for index regeneration.

Imagine, for example, that you have configured two identical VuFind servers: one for staging, and one for production. As long as both servers are running exactly the same Solr version with exactly the same schema, you could follow these steps to perform a minimal-disruption reindex process:


1.      On the staging server, reset your index as described in 6.3.1, and reindex all of your records as described in chapters 3 and 11.
2.      Copy the $VUFIND_HOME/solr/vufind/biblio directory on the staging server to a temporary location on the production server. The rsync command is a good way to do this – e.g., on the staging server, run: :code:`rsync -r $VUFIND_HOME/solr/vufind/biblio user@production-server:/tmp/` (in this example, note that user@production-server should be replaced by a valid username and valid server name).
3.      Stop Solr, move the new index into position, and then start Solr again. This will require a minimal amount of downtime, but it should be a matter of seconds or minutes rather than the longer period the full reindex process would have taken. The command for this might look something like this: :code:`systemctl stop solr ; mv $VUFIND_HOME/solr/vufind/biblio /tmp/biblio_old ; mv /tmp/biblio $VUFIND_HOME/solr/vufind/ ; systemctl start solr` (this four-part command stops Solr, moves the current (old) Solr core directory into the /tmp directory so you can get it back if you need to, then moves the new (reindexed) Solr core directory into position from the place in /tmp where we rsynced it, and finally starts Solr again… by stringing all of the commands together with semi-colons, we ensure that they run one after another without pausing, further minimizing any downtime).

This example procedure still requires a fair amount of manual effort, and is a rather crude demonstration of the possibilities of Solr. Solr has built-in replication capabilities that can be used to move indexes between servers automatically, with no downtime. The Solr Cloud feature offers even more powerful possibilities. To learn more about these features, see the Solr documentation (https://lucene.apache.org/solr/guide/).

6.4 Automating the Indexing Process
-----------------------------------

If you are using VuFind with an Integrated Library System, it is likely that your records will be changing regularly as new items are cataloged and old ones are weeded. You will want to keep your VuFind index up to date. Unfortunately, every ILS is different, and documenting the automation process for all of them in this book would be impractical. However, this section highlights some of the common tasks and steps you will need to understand to support automation.

Many VuFind libraries run a daily cron job which updates the index in the middle of the night, when activity is low. This cron job script should accomplish a few things:

1.      Retrieve new records from the ILS. In some cases, it may be possible to use OAI-PMH (see chapter 10); in other situations, it may be necessary to run an ILS-specific command-line script to extract records changed since the last run of the cron job. No matter how the records are obtained, they should be loaded into the index using the standard indexing tool as described in 3.2.

2.      Delete removed records from the index. When OAI-PMH is supported, this will be taken care of as part of that process. Otherwise, it may be necessary to obtain a list of deleted records in a different way, and then use VuFind’s $VUFIND_HOME/util/deletes.php script to remove them from the index.

3.      Delete suppressed records from the index. When working with an ILS that allows suppression of bib records, the $VUFIND_HOME/util/suppressed.php script can be used to automatically purge suppressed records from the index, assuming that VuFind’s connector to your ILS supports the necessary functionality.

4.      Optimize the index. After finishing updates to Solr, it is a good idea to run $VUFIND_HOME/util/optimize.php to ensure that your spellcheck index is fully up to date.

5.      Regenerate alphabetic browse indexes. If you are using VuFind’s alphabetic browse feature, you should run the $VUFIND_HOME/index-alphabetic-browse.sh script to ensure that browse indexes are up to date.

For more details and some real-world examples, see the Automation page of the VuFind wiki: https://vufind.org/wiki/administration:automation.

6.5 Other Important Automated Tasks
-----------------------------------

During the course of day-to-day operation, VuFind generates a significant amount of data that is needed for the short term but which should be cleaned up periodically to save storage space. This information includes user session data, search histories, and authentication tokens. The sections below explain the purpose of this data and how to clean it up when it is no longer needed.

6.5.1 Expiring Searches
_______________________

Every time a user performs a search in VuFind, a row is written to a search table in VuFind’s database. This table allows users to view their search history, and to save some of their searches for long-term use. However, when user sessions expire, many of these search history rows become orphaned and are no longer useful. If left unchecked, these obsolete database rows can grow significantly, wasting large amounts of disk space and impacting system performance. Fortunately, VuFind ships with a simple utility to clear them out. You can simply run:

.. code-block:: console

   php $VUFIND_HOME/public/index.php util expire_searches

to clear out old searches. It is strongly recommended that you run this command as part of a regular cron job to keep things under control.

6.5.2 Expiring Sessions
_______________________

VuFind also uses PHP sessions to store short-term user data (such as their login status). VuFind offers several options for where to store user sessions, such as on disk, in the database, or in a system like Memcached or Redis. The [Session] section of $VUFIND_LOCAL_DIR/config/vufind/config.ini documents all of the options and related settings.

Like stored searches, session data can build up over time, and while PHP is supposed to help clean this up for you, you may need to supplement PHP’s efforts with some work of your own to be sure things remain under control. If you are using file-based sessions, for example, you may wish to write a cron job to monitor the directory containing the session files and delete those that have not changed in a few days. If you are using database-based sessions, there is a command-line utility similar to the “expire_searches” tool that you can use:

.. code-block:: console

   php $VUFIND_HOME/public/index.php util expire_sessions

6.5.3 Other Expiry Tools
________________________

VuFind includes a couple of optional features that may require additional cleanup.

If you use the optional “email authentication” feature (which allows users to log in by clicking on a link in an email sent to them), you may need to periodically clean up the table of pending authentication hashes:

.. code-block:: console

   php $VUFIND_HOME/public/index.php util expire_auth_hashes
 
If you use Shibboleth authentication and the “single logout” feature, you may need to periodically clean up data used to track external user sessions:

.. code-block:: console

   php $VUFIND_HOME/public/index.php util expire_external_sessions

Over time, it is possible that additional features will be introduced which will require similar cleanup actions. You can always get a summary of VuFind’s available command line utilities by running:

.. code-block:: console

   php $VUFIND_HOME/public/index.php

Looking through this for additional “expire” actions should reveal whether anything new has been added since this book was written.

6.6 Managing Code and Configuration
___________________________________

As you customize and configure VuFind, you will find yourself making changes to dozens of files in multiple directories – configuration files, theme templates, custom code, automation scripts, etc. It is strongly recommended that you consider using a version control system like Git to track all of these things. Git was introduced in section 2.3.1, and if you skipped that section earlier, it may be worth revisiting it now. Even a basic understanding of Git will empower you in several important ways, as detailed in section 2.3.1.3. The value of version control cannot be underestimated; taking the time to learn about it now can save you from much costlier problems down the road.

Additional Resources
--------------------

As noted above, you can find more information about starting Solr automatically on the Starting and Stopping Solr page in the VuFind wiki (https://vufind.org/wiki/administration:starting_and_stopping_solr). You can learn more about automatic index updates on the Automation page of the VuFind wiki (https://vufind.org/wiki/administration:automation). Some of the topics from this chapter are demonstrated in the video available here: https://vufind.org/wiki/videos:administering_a_vufind_server.

Summary
-------

Reliably running a VuFind server in production requires some additional configuration and maintenance. By utilizing operating system auto-start features, intelligently managing your Solr indexing processes, and regularly cleaning up expired data, you can ensure that your users have a reliable and uninterrupted search experience.


Review Questions
----------------
1.      What is the difference between the “systemctl start” and “systemctl enable” commands?
2.      What are two reasons why you might want to rebuild your Solr index?
3.      Name four different types of data that may require automated cleanup processes.

