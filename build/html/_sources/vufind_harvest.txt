Working with XML and HTML
*************************

Chapter 10. Content Harvesting
##############################

Learning Objectives
-------------------

After reading this chapter, you will understand:

•       How VuFind uses the OAI-PMH protocol.
•       How to load XML-based data into VuFind’s index using XSLT.


10.1 Understanding OAI-PMH
--------------------------

OAI-PMH stands for the Open Archives Initiative Protocol for Metadata Harvesting. It provides a relatively simple set of rules for exchanging metadata between computer systems using XML documents. The standard was developed in the early 2000’s, with the current version of the protocol (2.0) being finalized in 2002. While it shows its age in some ways (such as its exclusive reliance on XML formats), it is widely adopted and supported, making it a very useful tool for obtaining records to index into VuFind.

At its most basic, OAI-PMH allows a client machine to download all of the metadata records from a server machine. In addition to supporting harvesting of full collections, OAI-PMH also allows a client to harvest records incrementally, receiving a collection of records that have been added, deleted or changed since a particular date. Other features include the ability to request specific subsets of a collection (which are defined by the server providing the records – the protocol sets no guidelines for how sets are determined) and the ability to retrieve records in multiple formats (though only Dublin Core, the favored “least common denominator” XML metadata standard, is guaranteed to be universally supported).

VuFind includes tools for harvesting and indexing records from an OAI-PMH server. It also includes built-in functionality that allows it to act as an OAI-PMH server if you wish to provide your records to others. This chapter discusses VuFind’s harvesting and server support in more detail; chapter 11 will talk about what to do with harvested records once you have them.

.. code-block:: console

 <?xml version="1.0" encoding="UTF-8"?>
 <OAI-PMH xmlns="http://www.openarchives.org/OAI/2.0/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.openarchives.org/OAI/2.0/ http://www.openarchives.org/OAI/2.0/OAI-PMH.xsd">
   <responseDate>2020-03-25T14:46:53Z</responseDate>
   <request verb="ListMetadataFormats">http://www.doaj.org/oai.article</request>
   <ListMetadataFormats>
      <metadataFormat>
         <metadataPrefix>oai_dc</metadataPrefix>
         <schema>http://www.openarchives.org/OAI/2.0/oai_dc.xsd</schema>
         <metadataNamespace>http://www.openarchives.org/OAI/2.0/oai_dc/</metadataNamespace>
      </metadataFormat>
      <metadataFormat>
         <metadataPrefix>oai_doaj</metadataPrefix>
         <schema>https://doaj.org/static/doaj/doajArticles.xsd</schema>
         <metadataNamespace>https://doaj.org/features/oai_doaj/1.0/</metadataNamespace>
      </metadataFormat>
   </ListMetadataFormats>
 </OAI-PMH>




As you can see, this reveals support for not just the standard oai_dc format, but also for a locally-defined metadata format called oai_doaj.

uFind includes examples for a variety of common formats, but if you harvest a brand new metadata format, you will also be responsible for defining rules for indexing it into VuFind; this is discussed further in chapter 11.   

10.2.4 Working with Identifiers
________________________________

As discussed in section 3.5.2, it is very important for every record to have its own unique identifier; without IDs, we can’t index things into Solr in a useful way. Some of the metadata formats provided by OAI-PMH servers – especially the richer ones like MARC -- will already contain useful identifiers. However, it is fairly common that records will not contain unique identifiers, or that they will contain multiple identifiers that are not easily differentiated from one another.

VuFind includes examples for a variety of common formats, but if you harvest a brand new metadata format, you will also be responsible for defining rules for indexing it into VuFind; this is discussed further in chapter 11.

10.2.4 Working with Identifiers
_______________________________

As discussed in section 3.5.2, it is very important for every record to have its own unique identifier; without IDs, we can’t index things into Solr in a useful way. Some of the metadata formats provided by OAI-PMH servers – especially the richer ones like MARC -- will already contain useful identifiers. However, it is fairly common that records will not contain unique identifiers, or that they will contain multiple identifiers that are not easily differentiated from one another.

Fortunately, VuFind’s OAI-PMH harvester provides a simple solution to this problem. When the OAI-PMH protocol provides records, it sends not just the raw metadata, but also a header above the metadata that includes additional information. This header always includes a unique ID for every record. The harvester only saves the metadata itself, not the header, but several configuration options exist for injecting values from the header into the metadata as custom XML tags.

If you add “injectId = identifier” to your oai.ini configuration section, then unique IDs from the header will be added to an <identifier> tag inside the top-level tag of your harvested metadata records. The “identifier” tag is used by all of VuFind’s example indexing configurations, but if you wish to use a different tag name for some reason, you can just specify a different value in the configuration.

It is also worth noting that most OAI-PMH record identifiers are quite verbose – for example, “oai:doaj.org/article:311ce1ec3dea42d2a7db0c3de149d865.” It may be desirable to shorten them and/or remove certain characters in order to improve the readability of URLs and avoid other problems (for example, some web servers may require configuration adjustments to support identifiers containing slash characters). Fortunately, there are oai.ini settings to address this situation as well: the idSearch[] and idReplace[] settings can be used in combination with injectId to perform regular expression replacements on IDs before injecting them into metadata. Regular expressions are also briefly discussed in section 5.3.6; they provide a standard language for matching patterns in text, and can be very useful for transformations like this.

If we wanted to replace the “oai:doaj.org/” prefix with a more concise “doaj_” prefix, we could revise our example configuration from earlier to:

.. code-block:: console 
   
   [doaj]
   url = http://www.doaj.org/oai.article
   metadataPrefix = oai_dc
   injectId = identifier
   idSearch[] = "|oai:doaj.org/|"
   idReplace[] = "doaj_"


…and the desired ID transformation will take place when the records are harvested.

If you need to make multiple adjustments to IDs, or if you need to account for several different possible patterns, you can repeat the idSearch[] and idReplace[] lines to create a series of rules that will be applied sequentially to every record ID.

10.2.5 Grouping Records Together
_________________________________

The default behavior of the harvest tool is to create a separate XML file on disk for each metadata file harvested. This keeps things simple, and it can be useful since it makes it easier to isolate problem records (if an import fails, there is no question about which record in a given file caused the problem). However, when loading records, it can slow down the process, since the indexing tools you are using will have to reinitialize themselves for each record.

The harvest tool provides configuration settings that allow you to group records together into fewer files. If you add “combineRecords = true” to your oai.ini section, each page of records loaded from the server will be stored in a single file, wrapped up inside a <collection> tag. If you want to change the name of the wrapping tag, you can use the combineRecordsTag setting to specify a different tag name.

The combineRecords functionality is ideal for harvesting MARC records; the SolrMarc import tool already knows how to deal with <collection> tags in MARC-XML, and it will load the files correctly. If you are working with other types of XML, it may be necessary to modify some of VuFind’s provided example import rules to account for multiple records per file; many of them were designed to assume they would only receive one record at a time, though this may be made more flexible in the future.

10.2.6 Troubleshooting
_______________________

The harvest tool is designed to be able to resume after a problem, so if there is a network connectivity interruption or remote server outage, if you repeat the harvest command, it will attempt to resume from the last place where it left off.

A common problem with harvesting has to do with invalid data on the remote server. It is a fairly common situation that OAI-PMH servers do not fully validate the XML that they are generating, and sometimes they include incorrectly formatted or illegal characters that cause validation errors for the client.

VuFind’s harvest tool contains some settings that can help resolve persistent problems related to XML validation. If you add “sanitize = true” to your oai.ini section, VuFind will automatically strip out illegal characters. If you set the badXMLLog setting to a filename, VuFind will store more detailed information about problematic XML in this file, which may be helpful for troubleshooting the issue with the content provider. Finally, the sanitizeRegex[] is a repeatable setting which can be used to provide regular expressions defining characters and patterns to remove from incoming XML. This can usually be left at its default value, but if you run into special situations, this provides the ability to customize the cleanup logic.

10.2.7 The Stand-Alone Harvest Tool
____________________________________

VuFind’s OAI-PMH harvest tool is also available as a separate project; if you ever need to perform a metadata harvest but do not need the full weight of VuFind, it may be useful to download the separate tool, which is available at https://github.com/vufind-org/vufindharvest. The only differences between the stand-alone version and the version found in VuFind are the name of the directory containing the executable PHP code (bin instead of harvest) and the fact that the stand-alone tool does not automatically look for an oai.ini file, since it has no concept of $VUFIND_HOME or $VUFIND_LOCAL_DIR. Instead, you need to use the “--ini=filename.ini” command-line switch to specify your configuration file.

10.3 Open Source OAI-PMH Servers
--------------------------------

Many commonly-used open source tools (including DSpace, Greenstone, Koha and OJS, the Open Journal System) include OAI-PMH server capabilities, as do many public repositories of shared open data (such as the Directory of Open Access Journals, DOAJ). VuFind includes sample configurations for harvesting the most popular of these tools, and those configurations can often be easily adapted to support others. This makes VuFind an ideal tool for creating the search “glue” between an ecosystem of open tools.

10.3.1 DSpace
_____________

The DSpace repository software contains an OAI-PMH server that can be enabled (see the DSpace documentation at https://wiki.lyrasis.org/display/DSDOC6x/OAI for details). Several metadata formats are supported, and VuFind includes built-in example configurations for indexing both the simple oai_dc metadata as well as the richer DIM format.

10.3.2 Koha
___________
The open source Koha Integrated Library System provides a built-in OAI-PMH service, which can be turned on with a configuration setting (see the Koha manual at https://koha-community.org/manual/18.05/en/html/webservices.html#oai-pmh for details). Once activated, you can point VuFind’s harvester at Koha using the marcxml metadataPrefix in order to retrieve records suitable for indexing with SolrMarc as described in chapter 3. Note that you can batch-load harvested MARC records using the harvest/batch-import-marc.sh script, which behaves very similarly to the harvest/batch-import-xsl.sh script described in section 11.3 below. Your import process will run more quickly if you harvest in groups as described in section 10.2.5.

10.3.3. OJS
___________

The OJS (Open Journal System) publishing platform includes built-in OAI-PMH support as well as a metadata plugin system which makes it possible to add support for custom metadata formats. VuFind includes sample import rules for both the Dublin Core and NLM (National Library of Medicine) formats.

10.4 VuFind’s OAI-PMH Server
____________________________

In addition to consuming OAI-PMH records, VuFind can also produce them. While VuFind’s OAI-PMH server is turned off by default, it can be activated by uncommenting and filling in the [OAI] section of config.ini. All of the available settings are described by comments in the .ini file; none are required (simply uncommenting the [OAI] section header is enough to turn on the server), but setting an identifier and repository_name are strongly recommended. Other settings exist to give you control over how your OAI-PMH server presents record sets and metadata formats.

Once set up, your OAI-PMH server base URL will be your VuFind URL with “/OAI/Server” appended; for example, Villanova University’s instance is https://library.villanova.edu/Find/OAI/Server. If you remove the “/Server” from the end of the page, you will be presented with a helpful form that you can use to test all of the standard OAI-PMH verbs.

It is very important to note that VuFind’s OAI-PMH server will only work correctly if you turn on some optional indexing features; these are discussed below.

10.4.1 Record Change Tracking
______________________________

Because an OAI-PMH server needs to be able to provide incremental updates showing which records have been added, changed, or deleted, VuFind needs to store some additional information at index time in order to keep track of these details. This functionality is disabled by default, because it makes the indexing process slower; however, that cost is necessary to achieve the benefit of OAI-PMH server functionality (and also some other potentially useful behavior, like properly-sorted RSS feeds and the ability to filter search results by record age).

If you are only indexing MARC records, activating record change tracking is as simple as uncommenting the first_indexed and last_indexed lines in VuFind’s example marc_local.properties file (see section 3.4.3). If you are also indexing XML records, you will need to ensure that the records contain information about modification dates and that your import rules correctly populate the first_indexed and last_indexed fields in Solr.

For more information about record change tracking, see the relevant page in the VuFind wiki (https://vufind.org/wiki/indexing:tracking_record_changes).
(https://vufind.org/wiki/indexing:tracking_record_changes).

Additional Resources
--------------------

You can read more about OAI-PMH at the protocol’s official website 
(https://www.openarchives.org/pmh/). VuFind’s OAI-PMH harvest tool has its own project page (https://github.com/vufind-org/vufindharvest). The VuFind wiki also contains notes on OAI-PMH harvesting (https://vufind.org/wiki/indexing:oai-pmh) and server functionality (https://vufind.org/wiki/indexing:tracking_record_changes#oai-pmh_server_functionality).

Summary
-------

The OAI-PMH protocol provides a common standard for sharing metadata. VuFind can take advantage of the protocol as both a consumer and a producer in order to pull together records from multiple systems and share its collection with others.

Review Questions
----------------

1.      What are the most important features of the OAI-PMH protocol?
2.      What are five commonly-used systems that provide OAI-PMH support?
3.      What configuration settings are required to allow VuFind to work as an OAI-PMH server?



