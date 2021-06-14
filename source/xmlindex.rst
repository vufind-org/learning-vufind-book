#########################
Chapter 11. XML Indexing
#########################

Learning Objectives
-------------------

After reading this chapter, you will understand:

•       The purpose of XSLT.
•       How to extend XSLT with custom functions.
•       How to use XSLT to index XML records.
•       How to import batches of records harvested with OAI-PMH.

11.1. Understanding XSLT
------------------------

XSLT stands for “eXtensible Stylesheet Language Transformations.” It is a declarative programming language used for defining rules to transform one XML document into a different XML document. An XSLT “program” is actually itself an XML document containing rules for selecting elements from a source document and using them to construct a target document. Since many metadata formats are represented using XML formats, and since Solr uses an XML-based format for updating its index, XSLT serves as a useful tool for indexing metadata into VuFind’s Solr index.

Several versions of the XSLT standard have been developed over the years, with version 3.0 being the most recent as of this writing. However, the PHP language only supports the original 1.0 version of XSLT; thus, by extension, VuFind also uses XSLT 1.0 for its built-in XML indexing tools. This is limiting in some ways, since later versions of XSLT added more richness to the language; however, most of these limitations can be overcome through one of XSLT’s most useful features: extensibility. The PHP XSLT interpreter allows you to register custom PHP functions so that they can be called from within an XSLT sheet. Thus, if a capability is missing from XSLT itself, you can overcome the limitation by delegating to custom PHP code.

Writing your own XSLT sheets and custom PHP functions is beyond the scope of this book; fortunately, for many common systems, you will not have to, since VuFind provides sample indexing configurations for several popular systems (and many of these examples can be adjusted to suit other systems with only minor changes). With an understanding of VuFind’s command line tools and even a rudimentary understanding of how XSLT works, you should be able to accomplish quite a lot.

If you want to gain a deeper understanding of the XSLT language, numerous helpful books and online articles exist. Just be sure to check which version of the standard is being discussed, since articles about XSLT 2.0 and newer may include some information that will not apply to VuFind’s basic XSLT 1.0 interpreter. Doug Tidwell’s XSLT, published by O’Reilly, is a thorough guidebook and reference; the first edition of the book (published in 2001) covers XSLT 1.0, while the second edition (published in 2009) discusses both 1.0 and 2.0.

11.2 VuFind’s Command-Line XSLT Indexer
---------------------------------------

VuFind’s XSLT indexing tool is found at $VUFIND_HOME/import/import-xsl.php. It can be run from the command line like this:

.. code-block:: console

   php $VUFIND_HOME/import/import-xsl.php <XML_file> <properties_file>

In this command, <XML_file> is the path to an XML file to index, and <properties_file> is the name of an indexing configuration file. For <properties_file>, you only need to specify a filename, not a full path; that filename will be searched for first in $VUFIND_LOCAL_DIR/import, and then in $VUFIND_HOME/import, allowing you to override configuration files through your local configuration directory following VuFind’s usual conventions.

The properties file specifies which XSLT sheet will be used to perform the transformation; example XSLTs are found in $VUFIND_HOME/import/xsl and can be overridden in the equivalent $VUFIND_LOCAL_DIR subdirectory as needed. The file also specifies which custom PHP functions will be made accessible to the XSLT code, and sets up parameters (like your institution name) that will be passed along to the XSLT.

All of the example properties files found in $VUFIND_HOME/import contain detailed comments explaining the various settings; you should look at one of these to find the latest up-to-date information on available options.

The import-xsl.php command accepts an optional --test-only switch, which puts it into test mode. In test mode, the result of the XSLT transformation is displayed on screen, and no data is sent to Solr. This is extremely useful for troubleshooting, testing and developing XSLT without potentially loading bad data into your index. For example, if you wanted to see what would happen if you applied the default OJS transformations to an XML record found in $VUFIND_LOCAL_DIR/harvest/ojs/record1.xml, you could run this command:

.. code-block:: console

    php $VUFIND_HOME/import/import-xsl.php --test-only $VUFIND_LOCAL_DIR/harvest/ojs/record1.xml ojs.properties

The result might look something like this:

.. code-block:: console

   <?xml version="1.0" encoding="utf-8"?>
    <add xmlns:oai_dc="http://www.openarchives.org/OAI/2.0/oai_dc/" xmlns:dc="http://purl.org/dc/elements/1.1/" xmlns:php="http://php.net/xsl" xmlns:xlink="http://www.w3.org/2001/XMLSchema-instance">
     <doc>
     <field name="id"/>
     <field name="record_format">ojs</field>
     <field name="allfields">The Useful Uselessness of the Humanities Roochnik, David Villanova University 2008-01-01 application/pdf http://expositions.journals.villanova.edu/article/view/82 Expositions; Vol 2, No 1 (2008); 19-26 North America Contemporary Authors who publish with this journal agree to the following terms:Authors retain copyright and grant the journal right of first publication with the work simultaneously licensed under a Creative Commons Attribution License that allows others to share the work with an acknowledgement of the work's authorship and initial publication in this journal.Authors are able to enter into separate, additional contractual arrangements for the non-exclusive distribution of the journal's published version of the work (e.g., post it to an institutional repository or publish it in a book), with an acknowledgement of its initial publication in this journal.Authors are permitted and encouraged to post their work online (e.g., in institutional repositories or on their website) prior to and during the submission process, as it can lead to productive exchanges, as well as earlier and greater citation of published work (See The Effect of Open Access).</field>
     <field name="institution">My University</field>
     <field name="collection">OJS</field>
     <field name="format">Online</field>
     <field name="author">Roochnik, David</field>
     <field name="author_sort">Roochnik, David</field>
     <field name="title">The Useful Uselessness of the Humanities</field>
     <field name="title_short">The Useful Uselessness of the Humanities</field>
     <field name="title_full">The Useful Uselessness of the Humanities</field>
     <field name="title_sort">useful uselessness of the humanities</field>
     <field name="description"> </field>
     <field name="publisher">Villanova University</field>
     <field name="publishDate">2008</field>
     <field name="publishDateSort">2008</field>
     <field name="url">http://expositions.journals.villanova.edu/article/view/82</field>
    </doc>
    </add>

11.3 Batch-Loading XML
----------------------
 
While indexing a single record is useful (especially when developing and testing a new set of import rules), it is much more common to want to ingest a batch of records all at once (such as after performing an OAI-PMH harvest as discussed in the previous chapter). Fortunately, VuFind includes a script to automatically ingest all of the XML files in a directory. It is used like this:

.. code-block:: console

   $VUFIND_HOME/harvest/batch-import-xsl.sh <harvest_subdirectory> <properties_file>

The <harvest_subdirectory> parameter is the name of a directory found under either $VUFIND_LOCAL_DIR/harvest or $VUFIND_HOME/harvest (following the usual VuFind pattern of checking the local directory first). The <properties_file> parameter specifies a configuration filename, exactly as described for the single-file importer in section 11.2.

When you run the script, it will create a “processed” subdirectory under <harvest_subdirectory>. It will index XML files from <harvest_subdirectory> one at a time, moving them into the “processed” subdirectory when they are successfully imported. Any files that fail to load correctly will not be moved, so you can troubleshoot them at the end of the process. If you ever want to re-index your records, you can simply move the files back out of the processed folder and into the main <harvest_subdirectory>.

If you performed an OAI-PMH harvest, you may also have a number of files in your harvest directory with names ending in “.delete,” tracking records that have been deleted from the source repository. There is a $VUFIND_HOME/harvest/batch-delete.sh script which will take care of removing these deleted records from your Solr index; it takes a single <harvest_subdirectory> parameter and behaves exactly the same as batch-import-xsl.sh in terms of moving files to the processed directory, etc.

Additional Resources
--------------------

The XSLT 1.0 standard used by VuFind can be found at https://www.w3.org/TR/xslt-10/. VuFind’s wiki page discussing XML indexing can be found at https://vufind.org/wiki/indexing:xml. A video about XML indexing can be found at https://vufind.org/wiki/videos:indexing_xml_records.

Summary
-------

VuFind includes tools to leverage XSLT 1.0 to index XML records into Solr. Separate configuration files and XSLT definitions can be created for importing different XML formats. A “test-only” mode makes it possible to preview transformations without modifying Solr prematurely. A batch loading script makes it possible to process folders filled with XML files (such as those produced by the OAI-PMH harvest tool discussed in chapter 10).

Review Questions
----------------

1.      What is XSLT and how does VuFind use it?
2.      How can you see the result of a record’s XSLT transformation without actually indexing that record into Solr?
3.      What command is used for batch-loading harvested XML records, and what parameters does it need?
