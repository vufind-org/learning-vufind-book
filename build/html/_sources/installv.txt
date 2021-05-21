Installing vufind
*****************

Chapter 2. Installing VuFind
############################

Learning Objectives
-------------------

After reading this chapter, you will understand:

•  The software requirements for installing VuFind.
•  The basic VuFind installation process.
•  Alternative options for installing VuFind such as Git and Vagrant, and reasons why you might wish to use them.

2.1 System Requirements
-----------------------

VuFind relies on several other pieces of software to do its work, and you will need to have access to these in order to use it. Fortunately, all of VuFind’s dependencies are free and supported by a wide variety of operating systems. Linux is the preferred operating system for VuFind installation, but the software can also be installed successfully on Windows and MacOS.

Before installing VuFind, you will need to install four key components:

• The PHP language.
• A relational database for storing user and session information; MySQL or MariaDb are recommended, as they are easier to use for basic use cases, but the software also supports PostgreSQL if necessary.
• A web server for exposing the VuFind interface to the Internet; Apache is strongly recommended.
• A Java Development Kit (JDK) for running VuFind’s Solr index, and the SolrMarc software used for indexing MARC records. If you will not be working with MARC records, you can use the lighter-weight Java Resource Environment (JRE), since the JDK is only a requirement of the SolrMarc indexer. If you will not be maintaining a local index (a rare situation, but possible in some cases), you will not need Java at all.

The specific versions required will depend on the VuFind version you are installing; details can be found on the requirements page of the VuFind website (https://vufind.org/wiki/installation:requirements).

2.2 Basic Installation
----------------------

The easiest way to install VuFind is to use a Debian package downloaded from the VuFind website (https://vufind.org/vufind/downloads.html). Debian packages are a method of distributing software for the Debian family of Linux distributions, which includes the popular Ubuntu flavor. The advantage to using a Debian package is that it will automatically install software prerequisites (PHP, MySQL, etc.) as well as complete some basic setup tasks, allowing you to begin working with the software after running only a few commands. Even if you are not currently working in a Debian environment but wish to evaluate the software, you can create a virtual machine using a tool like VirtualBox, install a clean copy of Ubuntu onto it, and follow these instructions from there.

If you wish to install VuFind in a different environment, consult the wiki page described above under “Additional Resources” and follow the instructions there.

2.2.1 Installing the Package
____________________________

First you need to download the .deb package; you can find the URL for the latest version of VuFind (along with historical releases) at https://vufind.org/vufind/downloads.html, and from a Linux command prompt, you can download it with the wget command; for example:

.. code-block:: console

  wget https://github.com/vufind-org/vufind/releases/download/v6.0.1/vufind_6.0.1.deb

Once the file is downloaded it, you can install it with:

.. code-block:: console

   sudo dpkg -i vufind_6.0.1.deb


Of course, you should substitute the appropriate filename if you have downloaded a different version of VuFind than the example 6.0.1).

This process will fail with an error, but this is expected: the problem is that you have not yet installed all of the software that VuFind needs; fortunately, this can be automatically fixed with this command:

.. code-block:: console

   sudo apt-get install -f

It will take some time to download and configure all of the components, but when the process completes, VuFind will be installed and ready for configuration!

Note that in many Debian-flavored distributions, a bit of extra configuration is needed to set up permissions so that VuFind can connect to the MySQL or MariaDB database software. Details vary from version to version; see the “Important Notes” in the Ubuntu installation wiki page for more details: https://vufind.org/wiki/installation:ubuntu 

If you get stuck or encounter a problem that is not addressed by the documentation, remember that you can reach out the community for support. See https://vufind.org/vufind/support.html for communication options.

Once successful, the Debian package install will have automatically done a few things for you, including building an Apache configuration to make VuFind accessible through a web browser, adjusting file permissions so that VuFind can write to its cache and update its own configuration files, and setting up some useful environment variables ($VUFIND_HOME and $VUFIND_LOCAL_DIR, which will be discussed further in section 3.3 below). There is a bit more manual work for you to do, however.

2.2.2 Starting Solr
___________________

VuFind’s default search functionality is powered by Solr, an open source indexing tool (discussed in much more detail in chapter 5). Because of its importance, VuFind’s installation process will complain if your Solr index is not running. If you do not plan to use Solr, you can ignore this message; however, if you want to be sure you see a full screen of success messages, you can start Solr now. This is simply a matter of switching to the VuFind directory and running the appropriate start command:

.. code-block:: console

   cd /usr/local/vufind
   ./solr.sh start

Solr can be configured to start automatically; this is discussed later in section 6.1.

If you receive warning messages or have other problems, you may wish to consult the wiki page on starting and stopping Solr: https://vufind.org/wiki/administration:starting_and_stopping_solr

2.2.3 Initial Configuration
___________________________

Open a web browser, and point it to http://localhost/vufind/Install -- this should open up a web page showing a number of setup steps. (Note that if you are installing VuFind on one computer and accessing a web browser on a different computer, you should replace “localhost” with the hostname of the VuFind system, and make sure that no firewalls are preventing the two machines from communicating over HTTP).

For each item showing a “Failed” status, click on it and follow the on-screen instructions to resolve the problem; once an issue is fixed, you can click the “Auto Configure” breadcrumb to return to the list.

Some potentially helpful notes:

•       As noted earlier, VuFind can connect to a variety of integrated library systems and library services platforms; by default, it simulates this connection with a “Sample” connector that returns fake data. The installer will warn you about this and offer you the option to configure a real ILS driver. If you do not plan to use an ILS at all, you can select the “NoILS” driver (see section 4.5.1.3), which will disable ILS functionality. If you are not ready to make this decision, you can safely ignore it for now; the setting can be easily changed later.
•       Setting up VuFind’s database can be the most challenging part of the installation process, because database security settings can prevent the automatic configuration from working. As mentioned above, the wiki installation documentation should have notes on the latest options for working around common problems.
•       Once everything is configured correctly, you should change file permissions on your configuration directory so that VuFind can no longer rewrite its own configurations; this will reduce the chances of accidental or malicious damage to your settings. The installer will provide guidance on how to do this once configuration is complete.

Once configuration is completed, you should have a fully functional VuFind instance operating at http://localhost/vufind on your system. Of course, there are no records in the system yet, so every search will come up empty. Chapter 3 will help resolve this problem, but first, it is worth learning about some alternative options for installing and managing VuFind.

2.3 Other Installation Options
------------------------------

While installing VuFind as a package is a reasonably straightforward way to manage the software, it may not be the best way to manage it in the long term, especially if you are a software developer. You may find it preferable to use Git to track changes and more easily perform updates, and you may wish to use Vagrant to quickly test the software’s performance in different environments without having to configure them yourself. This section describes the possible roles of these tools in VuFind installation and management.

2.3.1 Git
_________

2.3.1.1 Introduction to Git
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Git is distributed version control software, which is used by the VuFind community to manage development of the software. Git is a widely-used tool in open source, and a valuable asset if you are a software developer. Even for non-programmers, a basic understanding of Git can be helpful for deployment and upgrading of software.

The “version control” portion of “distributed version control” refers to Git’s primary function: tracking changes in software over time. As programmers add or change functionality, they “commit” these changes to Git’s history. This makes it possible to look back through the development of the software, identifying which programmers made which changes and reading their explanations of why those changes were made. When bugs are found, this makes it possible to identify which versions are affected. When mistakes are made, it is possible to roll them back. The software also supports multiple “branches” containing the code in different states of development; by “checking out” a branch, a Git user can instantly change the files on their disk to reflect a particular version of the code. Branches allow developers to work on multiple features at the same time, and test them independently; when work on a branch is completed, it can be “merged” back into the “master” branch, where the latest version of the code resides. When the code is deemed stable enough for an official release, the appropriate Git commit can be “tagged” with a version number, and these tags can be “checked out” just like branches, making it possible to quickly switch between different versions of the software for the purposes of testing and upgrading.


The “distributed” part of “distributed version control” refers to the fact that every user of Git creates their own “clone” or “fork” of the software repository that they are working with. They end up a full copy of all of the history and changes, to which they can add their own commits, branches and tags. This is a significant difference from earlier version control systems like Subversion, which relied on a single shared server to hold all of the change history, which made it more difficult for large groups of developers to work independently of one another. Git comes with tools for “pushing” and “pulling” changes between repositories, so users can work independently with their local repositories without having to worry about what others are doing, and then they can share their work “upstream” when it is in an appropriately polished state.

2.3.1.2 Installing VuFind with Git
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To install VuFind using Git, you first need to clone the official VuFind Git repository. If you wish to install the software in the default /usr/local/vufind directory, you could do it like this:

.. code-block:: console
   
   mkdir -p /usr/local/vufind
   cd /usr/local/vufind
   git clone https://github.com/vufind-org/vufind.git 

Git will give you all of VuFInd’s code, but nothing else; you will be responsible for installing all of the software that VuFind depends upon – both the requirements described in section 2.1, as well as the package’s Composer dependencies.

One simple way to install VuFind’s software requirements is to install the Debian package as described above. After the package and its dependencies have been installed, you can empty out the /usr/local/vufind directory and use Git to recreate the files (or you can leave the Debian installation alone, and use Git to install a separate copy of VuFind elsewhere on your system).

To install VuFind’s Composer dependencies, first install Composer (see https://getcomposer.org for instructions) and then, making sure you are in the directory where VuFind was cloned, run:

.. code-block:: console

   composer install

To learn more about Composer, see the accompanying sidebar.

2.3.1.2.1 Sidebar: About Composer
"""""""""""""""""""""""""""""""""

In open source development, it does not make sense to write new software if there is already a good component that can be reused. Most software packages of any complexity depend on many other projects to perform common tasks, and VuFind is no exception.

Managing these software dependencies can become complex, because components change over time, and it is important to receive updates to fix bugs while avoiding “backward compatibility breaking” changes that might cause problems. Most modern programming languages use tools to manage this process, and Composer is the preferred tool for PHP.

VuFind includes a file called composer.json, which lists all of VuFind’s dependencies, and the versions of those dependencies that are compatible with the rest of the code. Running the “composer install” command reads this file, downloads all of the relevant packages, and installs them into a subdirectory called “vendor.”

Most VuFind users do not need to concern themselves with this process, but if you plan to become more involved in the software development process, understanding this will be helpful.

Also note that if you install VuFind from a Debian package, or if you download a .tar.gz or .zip file from the website, the vendor directory is already populated for you, and you will not need to worry about Composer at all; this is only a necessary step when you are installing from Git.

2.3.1.3 Reasons for Using Git
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
There are several reasons why you may wish to consider using Git, most of which have been alluded to above:

•       By creating a local Git clone, you can create a branch representing your installed version of VuFind, and you can commit your local configurations to that branch. This will allow you to document the history of your changes to your settings, identifying when decisions were made, and more easily undoing changes that cause problems.
•       If you plan on managing VuFind on multiple servers (for example, development, staging and production environments), you can create branches for each environment, and merge changes between them. You can use the “push” and “pull” features of Git to deploy changes between servers.
•       You can more easily upgrade VuFind by pulling updates from the upstream repository and merging them into your local branches; once workflows are established, this can actually be easier than trying to upgrade Debian packages or manually deploy from .tar.gz or .zip files. Scripting can be used to help automatically upgrade your configurations and custom themes as well (see http://blog.library.villanova.edu/libtech/2015/07/23/automatically-updating-locally-customized-files-with-git/ for more information).
•       If you wish to participate in VuFind’s development, using Git is almost a necessity for sharing code with the rest of the community.

If you find Git intimidating, you certainly do not need to understand it to make use of any of the other information in this book. However, it is a valuable tool, and one that you should consider investigating in the future. Many books and online resources are available to help explain Git in much greater detail than this small section can manage.

2.3.2 Vagrant
_____________

2.3.2.1 Introduction to Vagrant
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Vagrant is a tool for automating the creation of virtual machines.

A virtual machine (VM) is a simulated computer system that runs on a different computer system. Virtual machines are a useful tool for running one operating system inside another (for example, you can create an Ubuntu VM and run it on a Windows machine); they are also a useful way to “sandbox” software – i.e. run programs in a disposable environment where, if something goes wrong, they can do limited harm.

Vagrant allows you to create a file called “Vagrantfile” which defines a basic environment (such as a particular version of Ubuntu) and a series of steps to perform in that environment (such as installing extra software). Vagrant configuration also allows files to be shared between the “host” machine and the VM, and for exposing access to the VM in a controlled way.

Manually setting up a VM can be a time-consuming and labor-intensive process; Vagrant makes this mostly automatic. A single command can create and configure a VM, and another command can destroy it when you are finished using it.

2.3.2.2 Using Vagrant to Run VuFind
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Using Vagrant to run VuFind is quite simple. No matter what method you used to install VuFind, you will find a Vagrantfile in the directory where the software was installed. You can switch to that directory and run:

.. code-block:: console
 
    vagrant up


This command will take quite some time the first time you run it, as Vagrant has to download a base image for the operating system that the VM will use, and then go through the process of installing and configuring VuFind. In general, after you have started Vagrant once, starting it again in the future will take less time.


Once Vagrant has finished starting up, you can “log in” to the virtual machine by running:

.. code-block:: console

   vagrant ssh

This will take you to a command prompt inside the VM. The /vagrant directory in this context is actually a link to the host machine’s VuFind home directory (usually /usr/local/vufind). There is also a directory called /vufindlocal which will hold the VM’s configuration files, and which will only be visible inside the virtual machine.

While the Vagrant VM is active, you can access its VuFind web interface through http://localhost:4567 on your host machine. This is accomplished through Vagrant’s port remapping, which exposes the VM’s port 80 (the standard port used for sharing HTTP web content) to the custom port of 4567 (to prevent the VM from conflicting with the host machine’s normal operation).

You can temporarily pause the VM with this command:

.. code-block:: console

   vagrant halt

.. code-block:: console

   vagrant shutdown

After either a halt or a shutdown, you can bring the machine back up by repeating:

.. code-block:: console

   vagrant up

When you are completely finished with the machine and no longer wish to use it, you can free up disk space by completely destroying it.

.. code-block:: console

   vagrant destroy

2.3.2.3 Reasons for Using Vagrant
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There are several reasons that Vagrant can be a useful tool:

•       Sometimes, the version of VuFind you want to run may not be compatible with your local machine. For example, your PHP version may be too old. Vagrant will automatically install a compatible operating system, and allow you to experiment with the software without having to change or upgrade your host system. Of course, if you wish to run VuFind in production, you will eventually need to set up a compatible server – using Vagrant for a live system is strongly discouraged – but having the ability to test things without having to wait for full server deployment can save a lot of time.
•       You may wish to try a potentially disruptive change – for example, some new custom indexing rules. Using a Vagrant box gives you an environment where you can test the change without risking damage to your host machine, and then throw away the results when you are finished.
•       You may wish to test whether VuFind will be compatible with a particular platform. As long as that platform has a Vagrant image available, you can modify the default Vagrantfile to use a different base image, and then see what happens, without having to reinstall an operating system or set up a new machine.


In general, most VuFind users will not need to use Vagrant – but when these kinds of use cases come up, it can be a valuable and time-saving resource.

Additional Resources
--------------------
A video covering many of the topics in this chapter is available through the VuFind website (https://vufind.org/wiki/videos). The installation page of the VuFind wiki (https://vufind.org/wiki/installation) contains more detailed and fully up-to-date, step-by-step instructions for installing VuFind in a variety of environments. If the methods described above were not appropriate for your needs, this information should prove helpful.

Summary
-------
VuFind can be installed in a variety of ways, depending on your needs. For a quick, production-ready deployment, using the Debian package under Linux is a convenient option. More experienced users may prefer to handle the installation themselves using Git, and developers may find Vagrant a convenient way to evaluate and test the software without making any potentially risky changes to real systems.

Review Questions
----------------
1. Where can you find the most detailed and up-to-date VuFind installation instructions?
2. What kind of operating system do you need to take advantage of a Debian package installation?
3. Should you use Vagrant to install VuFind in a production environment? Why or why not?
4. What are some advantages of installing VuFind using Git?
5. Why does the VuFind project use Composer?


