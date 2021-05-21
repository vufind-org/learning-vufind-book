Configuring Vufind
******************

Chapter 4. Configuring VuFind
#############################

Learning Objectives
-------------------

After reading this chapter, you will understand:

•  How to find key VuFind configurations.
•  How to configure search types.
•  How to configure facets.
•  The reason for the existence of additional configuration files.


4.1 Introduction and Review
---------------------------

VuFind is designed to be highly configurable. The software tries to offer many options through configuration, so that you can integrate with other systems and adjust functionality and appearance with a minimum of extra coding.


VuFind’s default configurations can be found in the $VUFIND_HOME/config/vufind directory and can be overridden through VuFind’s local directory. If you are not yet comfortable with the way VuFind’s local directory works, please review section 3.3 before continuing. Remember that files found under $VUFIND_LOCAL_DIR/config/vufind will be loaded when they are present; otherwise, default versions from $VUFIND_HOME/config/vufind will be loaded instead. You should NEVER edit the files under $VUFIND_HOME; always copy them to $VUFIND_LOCAl_DIR first.

The majority of VuFind’s configuration files use the .ini format, in which settings are divided into bracketed [Sections] containing “key = value” pairs. All of these files can be edited with a regular text editor, and they are full of comments explaining the meanings of settings and providing examples. Browsing through them to get a sense of available options may help you discover useful features you were not previously aware of.

Comments in .ini files are similar to comments in the SolrMarc properties files (described in section 3.5.1) except that .ini comments start with a semi-colon (;) instead of a pound sign (#). As a result, watch out for settings that start with a semi-colon; these are examples, and will not actually have any effect on VuFind until they are uncommented through the removal of the semi-colon.

4.2 config.ini
--------------

VuFind’s primary configuration file is named config.ini. When you run VuFind’s web-based installer (as described in section 2.2), one of its steps will copy $VUFIND_HOME/config/vufind/config.ini into $VUFIND_LOCAL_DIR/config/vufind/ for you, and will adjust some of the settings automatically to reflect details like your instance’s URL, and randomly generated keys for secure data encryption.

The config.ini file is the place where VuFind looks for most of its global settings – configurations that impact the software as a whole, as opposed to narrower aspects impacting one integrated system or specialized feature. This file controls which page VuFind loads when a user first accesses it, which theme it uses to present its user interface, how users are logged in, and which Integrated Library System it connects to (if any). A comprehensive discussion of every setting in the file would be prohibitively long (and the comments in the file itself provide the most up-to-date narrative), but being aware of this file as a starting point for configuration will help you find the most important settings. You will often see references to this file in online documentation and elsewhere in this book.

4.3 searches.ini
----------------

The searches.ini file contains settings related to the way VuFind presents search options related to its Solr index. This file lists legal search options (such as author, title, subject) and sort options (such as date, title, relevance). This is also the place to configure recommendation modules, which provide context-sensitive information accompanying search results (see Chapter 14 for more details). If you want to change default behaviors, rearrange options in drop-down menus, or customize search results screens, this file is a good place to start. Users interested in customizing the way searches are constructed will also need to understand the searchspecs.yaml, which is discussed later in section 5.3.

Note that searches.ini is only important if you are using the Solr index for your searches. If you are using a different search method, such as a third-party API, there will be a different configuration file (such as Summon.ini, WorldCat.ini, etc.) which will contain similarly-structured settings that will impact searches from that environment. Most VuFind users incorporate Solr in some way, but if your use case does not require Summon, it is possible that you will never use searches.ini – but understanding its contents will still be useful because whatever backend you use probably relies on most of the same basic concepts.

4.4 facets.ini
--------------

The facets.ini file controls VuFind’s faceted search behavior – the lists of values that can be applied to filter search results, as well as pre-filtering options displayed on the advanced search screen, and lists of commonly-found values displayed on the default home page. If you want to add, disable, or rearrange values in any of these places, this file will allow you to effect those changes.


As with searches.ini, this file only affects searching of VuFind’s standard Solr index; if you are searching a different service, it will have similar facet settings in its own backend-specific configuration file.

4.5 Other Configuration Files
-----------------------------

There are many additional configuration files found in $VUFIND_HOME/config/vufind, which fall into three broad categories.

4.5.1 ILS Driver Configurations
_______________________________

As noted earlier, VuFind can connect to a wide variety of integrated library systems / library services platforms. Each of these requires special code (called an ILS driver) and configuration (a driver-specific .ini file) to enable communication between VuFind and the target system. There is a [Catalog] section in config.ini which determines which ILS driver VuFind will use. In nearly every case, there is an .ini file in the configuration directory matching the name of the driver, which will contain important settings that will need to be configured – often connection credentials of some sort to grant permission for VuFind’s access, and sometimes additional settings to specify which version of the software is being used, or to enable/disable optional behaviors.

In addition to the straightforward ILS drivers that correspond with existing commercial and open source projects, there are also some special ILS drivers.

4.5.1.1 The Demo Driver
^^^^^^^^^^^^^^^^^^^^^^^^
VuFind’s Demo ILS driver simulates all the functionality of a real ILS driver, but instead of connecting to a real system, it makes up fake results and stores them internally. This is primarily used for software development and testing, since it makes it possible to test and change VuFind’s core behavior without access to a third-party system. It may also be useful for evaluation purposes, to see how a feature will work if you add an ILS in the future. There is no reason to use this driver in a production system, however.

The Demo.ini configuration file controls the behavior of the Demo driver by turning features on or off, or controlling whether or not the driver should simulate system failures for testing purposes.

4.5.1.2 The MultiILS Driver
^^^^^^^^^^^^^^^^^^^^^^^^^^^
The MultiILS driver allows a single copy of VuFind to communicate with multiple ILS drivers. By putting the proper configuration into MultiILS.ini and by indexing records with special ID prefixes, you can set things up so that you can index records from multiple libraries, and have VuFind communicate with the appropriate systems when fetching availability information, placing holds, etc.

The MultiILS setup is quite complicated, and is only rarely needed (in use cases such as union catalogs), so a detailed discussion is beyond the scope of this book. For more detail, see the appropriate wiki page (https://vufind.org/wiki/configuration:ils:multibackend_driver).

4.5.1.3 The NoILS Driver
^^^^^^^^^^^^^^^^^^^^^^^^^
The NoILS driver disables some or all of VuFind’s ILS-specific behavior, and it can also replace some functionality normally delegated to an ILS with data retrieval from the Solr index. There are two different use cases for this driver:

1.      By using the loadNoILSOnFailure setting in config.ini, VuFind can be configured to load the NoILS driver instead of the regularly configured driver when a problem is encountered. You can then set the NoILS.ini file into “ils-offline” mode to display a message about temporarily unavailability of ILS-related functionality. This is useful to give your users a better experience during planned or unplanned outages of your ILS.
2.      If you have no ILS at all, you can select NoILS as your driver and set the NoILS.ini file into “ils-none” mode, and this will ensure that VuFind hides functionality related to the ILS at all times.


Whether disabling ILS functionality until you are ready to pick a system, or as a safety net in case of failures, the NoILS driver is an important tool to be aware of when configuring VuFind.

4.5.2 Search Backend Configurations
___________________________________

As noted above, the facets.ini and searches.ini configurations are for Solr, but VuFind can integrate with a wide variety of other search systems. In VuFind terminology, the collection of code used for connecting VuFind to an external search system is referred to as a “search backend,” and most of these backends have corresponding .ini files for storing connection credentials, search/facet preferences, and other service-specific details.

Note that some search backends – such as those for Summon and WorldCat – require some connection credentials to be added to config.ini in addition to the backend-specific .ini files; these configurations have not been moved in the interest of backward compatibility with earlier releases of VuFind, but they may be relocated in future to make the configuration layout more uniform.

Apart from this one potentially confusing inconsistency, the search backend configurations have been intentionally designed to be as similar to one another as possible. Different systems use different conventions for naming data fields and specifying search types, so the configuration files can vary in significant ways, but the names and behaviors of settings have been kept the same as much as possible. If you learn how to configure one search backend, that knowledge should transfer to configuring the next one, should you need to set up multiple side-by-side search options, or should you migrate from one system to a different one in the future.

The subject of search backends is discussed in greater detail in chapter 15.

4.5.3 Feature-Oriented Configurations
_____________________________________

Some features of VuFind require especially complex configuration and/or are only used in very specialized situations, and putting all of those settings into the main config.ini files would make that file harder to read and work with. Thus, they have been split out into separate files. Some important examples include: combined.ini and searchbox.ini, which will be discussed in more detail in chapter 13; export.ini, which controls the ways in which users can download record data from the system; permissions.ini, which provides rule-based access control over some of VuFind’s features; and sitemap.ini, which controls the creation of sitemap files to assist search engine crawling (see also section 12.1).

Additional Resources
--------------------
For a more comprehensive and up-to-date list of configuration files, see the appropriate wiki page (https://vufind.org/wiki/configuration:files).

Summary
-------

VuFind is a heavily configuration-driven piece of software, and it includes what can be an intimidating number of configuration files. However, most users will only need to edit the main config.ini file, and a few additional configurations related to specific systems they integrate with, and particular features that they wish to customize. Because modified configurations need to be stored in VuFind’s local configuration directory, users will always be able to easily focus in on which files are important to their installation.

Review Questions
----------------

1.      What are the primary purposes of config.ini, searches.ini and facets.ini?
2.      What are two reasons you might wish to configure the NoILS driver?
3.      What does a semi-colon (;) in a configuration file mean?
4.      What are three feature-oriented configuration files (excluding searches.ini and facets.ini), and why would you want to use them?
5.      What is a search backend?




