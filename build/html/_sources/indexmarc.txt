Indexing Marc
*************

Chapter 3. Indexing MARC Records
################################

Learning Objectives
-------------------

After reading this chapter, you will understand:

•  How to load MARC records into VuFind’s Solr index.
•  How VuFind’s configuration files are organized and loaded.
•  How to customize VuFind’s MARC indexing process.

3.1 Obtaining MARC Records
--------------------------

The MARC record has been a long-enduring data exchange format for library records. Discussing MARC records in detail is beyond the scope of this text, but the Library of Congress’ Understanding MARC Bibliographic (https://www.loc.gov/marc/umb/) provides a good introduction. For the purposes of this text, suffice it to say that the format arranges data into numeric fields which are divided into alphanumeric subfields.

VuFind ships with powerful tools for indexing MARC records, making MARC one of the easiest formats to begin working with. Since most library systems are capable of exporting MARC records, if you have an integrated library system, there should be a straightforward way to extract all of its records for loading into VuFind. The details of this process will vary from system to system, but notes for a variety of commonly-used platforms can be found in the wiki (https://vufind.org/wiki/indexing:marc:export_notes).

If you do not have easy access to your own MARC records but wish to experiment with VuFind, there are also a variety of sample records included with the software under the tests/data directory. These sample records will be used in the examples throughout this chapter.

3.2 Running SolrMarc
--------------------

SolrMarc is an open source project for loading MARC records into Solr (a topic which will be discussed in much more detail in chapter 5). Every VuFind release includes a copy of SolrMarc and scripts for running it, so there is no need to install anything extra to get started. To load records into VuFind, you simply need to run the import-marc.sh script and specify the file containing MARC records (which may be either binary MARC format or MARCXML, as long as its filename ends in an appropriate .mrc or .xml extension). For example:

.. code-block:: console

   cd /usr/local/vufind
   ./import-marc.sh tests/data/journals.mrc


Note that your Solr index must be active when you run SolrMarc; see 2.2.2 above for instructions on starting Solr.

Once the indexing process is complete, you should be able to return to http://localhost/vufind in your web browser and submit the search form to see what you have indexed; submitting a blank search is a useful way to retrieve all records from the Solr index. Now that we have seen some records added to the system, let’s back up and look at how they got there.

SolrMarc maps MARC records into Solr using configuration files, and the defaults that ship with VuFind should meet most needs. However, most users will want to customize at least some fields, and in some situations, customizations will be necessary for successful record loading. Before getting into the details of SolrMarc customization, however, it is important to understand how VuFind stores and loads its configuration files.

3.3 Understanding VuFind’s “Local Directory”
--------------------------------------------

In section 2.2.1, it was mentioned that installing VuFind with a Debian package sets up two environment variables on your system: $VUFIND_HOME and $VUFIND_LOCAL_DIR. $VUFIND_HOME contains the full path to the directory containing the VuFind software (/usr/local/vufind by default); $VUFIND_LOCAL_DIR contains the full path to VuFind’s “local directory” (by default, /usr/local/vufind/local).

Throughout the rest of this text, when referring to the VuFind home or local directories in file paths, the shorthand variables will be used – so, for example, $VUFIND_LOCAL_DIR/config means /usr/local/vufind/local/config, and $VUFIND_HOME/config means /usr/local/vufind/config. As long as you have these environment variables set on your system, you can use them in commands. This means that if you choose to install VuFind in a non-default location, you can simply redefine the variables (usually by editing the file /etc/profile.d/vufind.sh), and the commands listed in this book will still work correctly.

The purpose of the local directory is to separate locally managed configuration files and data from the core VuFind code. When VuFind needs to load a configuration, it will first look for that file in the local directory; only if the file is missing from the local directory will VuFind load the default version from the home directory.


 there is a file matching $VUFIND_LOCAL_DIR/import/import.properties, it will be loaded; if not, $VUFIND_HOME/import/import.properties will be used instead.

When configuring VuFind, you should always copy configuration files into the local directory and edit them there; you should NEVER edit core configuration files.

When used correctly, the local directory approach has several advantages:

•  By looking in the local directory, you will know exactly which files you have customized. This can be especially valuable when troubleshooting problems, because you can easily compare your changes against the core defaults to see what has been changed, and you can even temporarily disable customizations by moving or renaming files.
•  VuFind’s automated upgrade process takes advantage of the separation between core and local files to help you automatically update your configuration files when you upgrade to a new version of VuFind.
•  It is possible to maintain multiple local directories containing different settings, and change VuFind’s behavior by changing the value of the $VUFIND_LOCAL_DIR variable. This can be useful, for example, when several libraries share a central index but want to have different user interfaces. This topic is discussed in much more detail in the wiki, on the Installing Multiple Instances page (https://vufind.org/wiki/installation:installing_multiple_instances).


The local directory can be a source of confusion for new VuFind users, since it is easy to accidentally edit the wrong version of a file if you are not careful. If you edit a configuration file and changes do not appear to take effect, make sure you are in $VUFIND_LOCAL_DIR and not in $VUFIND_HOME. Fortunately, VuFind does not require you to remember too many rules, but the rule of never editing core files is an important one that, if followed consistently, will make your VuFind experience run smoothly.

3.4 About SolrMarc’s Configuration Files
----------------------------------------

VuFind’s installation of SolrMarc relies on three key configuration files, all of which have default versions in the $VUFIND_HOME/import directory which you can override in $VUFIND_LOCAL_DIR/import. The first of them, import.properties, will be automatically customized for you with some important file paths as part of the VuFind installation process, so do not be surprised when you find your local import directory already populated with a file.

3.4.1 import.properties
_______________________

The import.properties file tells SolrMarc some of the most basic information it needs to function: where its other configuration files are located, the URL where Solr is running, and some advanced preferences. In most cases, the defaults created by VuFind’s installer will work correctly, and there is no need to edit this file. However, if you run Solr in a non-default way, or if you encounter problems with the processing of your MARC file, some of the settings in this file may need to be changed. The file contains comments explaining its contents.

3.4.2 marc.properties
_____________________

The marc.properties file is the key to SolrMarc’s behavior. It provides rules for extracting data elements from MARC records and storing them in named fields in the Solr index. These fields are used by VuFind for searching, faceting and record display; Solr will be displayed in much more detail in chapter 5, but the file should be understandable without detailed knowledge of Solr.

SolrMarc supports several different types of mappings:

•       Static text strings: if you always want to set a field to the same value, regardless of the contents of the MARC record, you can assign some double-quoted text to a field name, and SolrMarc will insert that value into every record that it indexes. This is used in the default configuration to set the “building” value of every record to “Library A” as an example.
•       Field specifications: SolrMarc contains its own special language for selecting MARC fields and subfields. Generally, this consists of number/letter combinations, like 035a to select subfield a of field 035, or 100abcd, to select the contents of the a through d subfields of field 100 as a single value. You can combine several of these selectors with colons to select a list of values from all of the specified fields; for example, 440ap:800abcdfpqt:830ap will select values from the specified subfields of the 440, 800 and 830 fields.
•       Custom functions: In some situations, selecting data for indexing requires more complex logic than simply selecting a set of fields and subfields. In these situations, a function can be written in the Java programming language, and this custom logic can be accessed in SolrMarc using the “custom” keyword. SolrMarc itself comes with several custom functions, and VuFind adds more. If you need to, you can also build your own, though that topic is beyond the scope of this book. If you want to examine the code for VuFind’s custom indexing functions, you can find them in the $VUFIND_HOME/import/index_java directory.

SolrMarc also provides a number of modifiers which can be added after field specifications or custom functions, which can filter or change the selected values. A very common one is “first,” which will filter down a set containing multiple values to just one value. This is useful in situations where multiple values may be present, but only one is needed.

This quick summary of SolrMarc functionality is intended to help you read and understand VuFind’s default configurations, but it only scratches the surface of the available functionality. For a much more detailed description of available options and their meanings, you can read the documentation available through SolrMarc’s wiki (https://github.com/solrmarc/solrmarc/wiki).

3.4.3 marc_local.properties
___________________________

The import.properties file tells SolrMarc some of the most basic information it needs to function: where its other configuration files are located, the URL where Solr is running, and some advanced preferences. In most cases, the defaults created by VuFind’s installer will work correctly, and there is no need to edit this file. However, if you run Solr in a non-default way, or if you encounter problems with the processing of your MARC file, some of the settings in this file may need to be changed. The file contains comments explaining its contents.

3.5 Customizing SolrMarc
------------------------

Most users of SolrMarc will want to make a few simple customizations; this section describes how to perform some of the most commonly needed changes.

3.5.1 Overriding Default Collection, Institution and Building Values
____________________________________________________________________

As noted above under 3.4.2, VuFind’s default indexing configuration includes some made-up values like “Library A” in the building field. “Catalog” in the collection field and “MyInstitution” in the institution field are other hard-coded values that most users will want to override with more appropriate values. Doing this is quite simple. First, if you do not already have a marc_local.properties file, create one by copying the default version into your local directory:

.. code-block:: console

   cp $VUFIND_HOME/import/marc_local.properties $VUFIND_LOCAL_DIR/import/

Next, use your editor of choice to edit the resulting $VUFIND_LOCAL_DIR/import/marc_local.properties file. You will see that it contains lines that look like this:

.. code-block:: console
 
   # Uncomment the following settings to insert appropriate values for your site:
   #collection = "Catalog"
   #institution = "MyInstitution"
   #building = "Library A"

Note that all of these lines start with a # character – the # symbol at the beginning of a line tells SolrMarc that these are comments intended for a human, and they should be ignored by the software. Lines such as these are said to be “commented out.” You can “uncomment” them by removing the # signs, and then SolrMarc will obey the instructions. For example, you could change them to look like this:

.. code-block:: console

   # Uncomment the following settings to insert appropriate values for your site:
   collection = "Online Catalog"
   institution = "VuFind University"
   building = "Main Library"

Once you have adjusted the settings to meet your needs, you must reindex all of your records (by re-running the import-marc.sh command as described in section 3.2). Remember, SolrMarc transforms records and loads them into Solr. Changing its configuration file will not have any effect on records that you loaded in the past; it will only change the way new records are loaded. Every change you make will require a full rebuild of the index.

3.5.2 Loading ID Values
_______________________

When indexing records, it is very important to make sure that the “id” field is filled in correctly. Every record in VuFind needs to have its own unique ID, and this should correspond with the bibliographic record identifier in your Integrated Library System, assuming that you are using one. This value will be used by VuFind to retrieve availability from your ILS and to construct unique record URLs, and by Solr to tell records apart. If you index two records with the same ID into Solr, the second record will overwrite the first one. This mechanism is what makes reindexing existing records behave correctly, but it can cause strange problems if the “id” field is not set up correctly.

In VuFind’s default configuration, the MARC 001 field is used to populate “id.” This will work correctly for many systems, but there are some that place the bibliographic identifier in a different place. For example, some methods of exporting records from the Koha ILS will put the appropriate identifier in field 999, subfield c. Thus, to index these records correctly into VuFind, you would have to establish and edit a local copy of marc_local.properties (as described under 3.5.1 above), and then add the line:

.. code-block:: console

   id = 999c, first

The “first” modifier is probably not strictly necessary, as no Koha record should be exported with more than one ID value. However, it adds a little bit of extra safety in case of an anomaly; without configuration to limit the id field to only one value, a record with multiple IDs would cause a failure in the indexing process, since VuFind’s Solr configuration is not set up to understand how to process a record with more than one ID.

Because of the special role of IDs in Solr indexing, it is also important to be careful about how you manage your index after changing the way IDs are determined. When you reindex records with different ID values, the new records will not overwrite the old ones, and you may end up with duplicates in your system. It is generally a good idea to empty out your index before reindexing when you change ID rules; the process for resetting a Solr index will be discussed below in section 6.2.


Additional Resources
--------------------

A video covering many of the topics in this chapter is available through the VuFind website (https://vufind.org/wiki/videos). The “Indexing MARC” page of the VuFind wiki (https://vufind.org/wiki/indexing:marc) contains additional details and advice that may be more in-depth and up-to-date than this chapter.

Summary
-------
SolrMarc provides a fast and powerful way of loading MARC records into your VuFind system, making them searchable by your users. It uses VuFind’s “local directory” mechanism to manage its configuration files. SolrMarc has a flexible built-in language that you can use to specify exactly how your records are mapped into VuFind’s index, and VuFind provides a reasonable default configuration that should provide a solid foundation to build upon.

Review Questions
----------------

1. What is VuFind’s “local directory,” and why should you use it?
2. What is the difference between marc.properties and marc_local.properties?
3. What will happen if you index two different MARC records that have the same value in the field used as Solr’s unique ID?
4. How can you change the values that display in VuFind’s “Collection” and “Building” facets in the search result screen?
5. What do $VUFIND_HOME and $VUFIND_LOCAL_DIR mean?
6. Your ILS places bibliographic identifiers in MARC field 997, subfield c. How do you tell SolrMarc to use this as Solr’s unique ID?

