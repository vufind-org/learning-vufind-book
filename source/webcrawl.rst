Part 4. Working with XML and HTML
*********************************

Chapter 12. Web Crawling with VuFind
####################################

Learning Objectives
-------------------

After reading this chapter, you will understand:
•       The XML sitemap standard.
•       VuFind’s full text indexing capabilities.
•       Configuration options for website search.


12.1 Understanding XML Sitemaps
-------------------------------

Search engines are a critical component of the Internet’s infrastructure, making it possible for users to locate the resources that they need. When first developed, search engines found content by “crawling” from link to link; this technique is still used and can be effective, but it has some limitations, especially for sites that are themselves more search-oriented than browse-oriented. It is very helpful for search engines to be able to easily get a list of all pages on a site without crawling so that they can more efficiently and effectively index the available content. The XML sitemap standard was developed to meet this need.

An XML sitemap is a document that lives on a web server – frequently using the standard path of /sitemap.xml – which contains a structured list of URLs available for crawling, supplemented with some additional metadata about frequency of updates, indexing priority, etc. Very large and complex sites may split their sitemaps into several files, which are all referenced from a top-level /sitemapIndex.xml file. The sitemap standard also interacts with the robots.txt standard (used to regulate the behavior of search engine crawlers) by providing a robots.txt directive to specify the location of any or all XML sitemaps. All of the details of the specification can be found at the standard’s website, https://www.sitemaps.org.

Many search engines will try to automatically locate XML sitemaps, but some also provide mechanisms for specifically reporting sitemaps to them; for example, Google’s Webmaster Tools can be used to request that Google crawls your sitemaps.

VuFind can serve as both a producer and consumer of XML sitemaps. This chapter will address both use cases.

12.2 Generating Sitemaps in VuFind
----------------------------------

As mentioned in section 4.5.3, there is a sitemap.ini file in VuFind’s standard configuration directory which can be used to configure where and how VuFind builds XML sitemaps. VuFind can, for example, build XML sitemaps documenting all of the records in your index, plus a top-level sitemap index that links to the VuFind-generated sitemap as well as additional sitemaps you specify (useful, for example, if you integrate VuFind with another content management system to build your overall website).

To create XML sitemaps for your VuFind instance, follow these steps:

1.)     If you have not already done so, copy $VUFIND_HOME/config/vufind/sitemap.ini to $VUFIND_LOCAL_DIR/config/vufind/ and edit the file to specify your preferences. The detailed comments included in the .ini file document the purpose and meaning of all of the options.
2.)     Run *php $VUFIND_HOME/util/sitemap.php* to generate the sitemaps.

As records are added or removed from your system, you will need to re-run the sitemap generator command to update the files; this is a good candidate for automation through a cron job, or as part of your regular indexing processes.

Be sure that the user running the sitemap generation process has appropriate permissions to write files into the directory where your sitemaps will be served from.

12.3 VuFind’s Web Crawler
-------------------------

VuFind is often used in combination with a content management system like Drupal or Concrete5; VuFind provides the search capabilities while the CMS provides static content. For example, a library might use VuFind for catalog searching, but have a CMS for providing lists of online resources, guides for researching particular topics, etc. In this kind of scenario, it is useful to be able to make all of the web content searchable through VuFind, so users can perform a full text search across the entire site. To support this use case, VuFind includes a separate Solr core designed for searching web pages. The default tool for populating this index relies on XML sitemaps to retrieve content. Fortunately, most commonly used platforms already support sitemap generation, and the format is so simple that it is possible to build them by hand (for a small number of pages) or develop tools to help build them (if you have some basic experience with script-writing).

Setting up web indexing in VuFind is a two-step process: first, you must configure a tool to extract searchable text from web pages; then, you need to configure VuFind’s web crawling tool. The following two sections discuss these steps in more detail.

12.3.1 Setting Up Full Text Indexing
____________________________________

In order to perform full-text searching of a web page, VuFind needs a way to extract all of the searchable text from the HTML document. Fortunately, several tools exist for extracting text from documents, and VuFind provides built-in support for integrating with two of these: Apache Tika, one of the most popular options, as well as Aperture, an older and lesser-used tool. Most users will want to use Tika, as it is better-maintained and supported.

To get set up, follow these steps:

1.)     Download the latest version of Tika’s runnable jar from https://tika.apache.org/download.html and put it somewhere on your server; for this example, we will assume you have chosen /usr/local/tika/
2.)     Copy $VUFIND_HOME/config/vufind/fulltext.ini to $VUFIND_LOCAL_DIR/config/vufind/.
3.)     Edit the new local copy of fulltext.ini. Uncomment the [General] section header and the “parser = Tika” lines to activate Tika support, and set the “path” setting under the [Tika] section to the full path to your downloaded jar file – e.g. /usr/local/tika/tika-app-1.24.jar.


Now, when VuFind needs to extract full text from a document, it will use Tika to do so. This is useful not only for website indexing, but also for extracting full text from documents referenced in metadata files indexed into your main biblio core. Tika supports many document formats, so it is useful not just for HTML documents but also for Microsoft Word files, PDFs, etc. For example, if you want to index the content of documents referenced in 856 fields in MARC records, you can now use SolrMarc’s custom getFulltext method (see example in $VUFIND_HOME/import/marc_local.properties). Similarly, if you want to index linked content in XML metadata, you can use VuFind’s custom harvestWithParser() method, as demonstrated by the example in $VUFIND_HOME/import/xsl/nlm_ojs.xsl.

12.3.2 Configuring and Running the Web Crawler
______________________________________________

Once full text indexing is configured, setting up VuFind’s web crawler is quite simple. Copy $VUFIND_HOME/config/vufind/webcrawl.ini to $VUFIND_LOCAL_DIR/config/vufind/ and edit the file. Simply add a “url[] =” line to the [Sitemaps] section for every sitemap URL that you wish to crawl. If your site has a sitemapIndex.xml, you can point to this single file and VuFind’s crawler will follow all of the links within it. You can turn on the “verbose” setting in the [General] section if you want to see more detailed output while the indexing process is running.


Once the configuration is set up, you can initiate the crawling process with the command:

.. code-block:: console
 
   php $VUFIND_HOME/import/webcrawl.php


This will index all of the pages in all of the sitemaps referenced in webcrawl.ini. This process can take a long time for large sites, since it has to download every web page in order to index it. When it finishes indexing new pages, it will delete any pages in the index that have ceased to exist since the last time the tool was launched. For this reason, you should never run the web indexer while your site is offline, since it could end up removing useful content from your index.

The webcrawl.php tool operates by applying an XSLT to the downloaded sitemap.xml files; it is actually a specialized version of the XML indexer described in chapter 11. If you need to make changes to the way pages are indexed (for example, to extract the content of a specific <meta> tag into a custom index field for faceting purposes), you can override and customize $VUFIND_HOME/import/sitemap.properties and/or $VUFIND_HOME/import/xsl/sitemap.xsl as needed. For an example of this type of customization, see the WordPress section of the article “The Triumph of David: A Case Study in VuFind Customization,” published in Annals of Library and Information Science v. 63, no. 4 and available online here: http://op.niscair.res.in/index.php/ALIS/article/view/14527.

12.4 Accessing and Customizing VuFind’s Web Search
--------------------------------------------------

Once you have finished indexing content, you can search your web index through VuFind’s separate Web search; the URL will be something like http://localhost/vufind/Web, assuming that http://localhost/vufind/ is your VuFind base URL.

If you wish to customize the behavior of the web search, there are several files that you can override as needed:

-  $VUFIND_HOME/config/vufind/website.ini – This file contains settings equivalent to the contents of searches.ini and facets.ini, but applied to the website index instead of the main biblio core. See sections 4.3 and 4.4 for more detail.
-  $VUFIND_HOME/config/vufind/websearchspecs.yaml – This file contains rules used for managing relevance ranking of search results, following the same format as the main searchspecs.yaml used by the biblio core. See section 5.3 for more detail. 
-  templates/RecordDriver/SolrWeb/result-list.phtml – This template file can be overridden within your theme to change the way individual web results are displayed in the search result list; the VuFind\RecordDriver\SolrWeb class can also be extended to add functionality as needed. For more on customizing record views, see chapter 9.

Unless you are planning on using VuFind exclusively for web searching, you will likely want to make it convenient for users to seamlessly search across both the web index and the main bibliographic record index. See chapter 13 for more on how to combine different types of searches using VuFind.

Additional Resources
--------------------

Notes on VuFind’s web crawling tools can be found on this wiki page: https://vufind.org/wiki/indexing:websites.

Summary
-------

XML sitemaps provide a useful way to publish lists of web pages for consumption by search engines. VuFind can produce its own sitemaps to make indexed content more visible in search engines, and it can consume external sitemaps to build its own searchable web page index as a complement to its core bibliographic record index.

Review Questions
----------------

1.      What is the difference between sitemap.xml and sitemapIndex.xml?
2.      What configuration files do you need to edit in order to set up web indexing in VuFind?
3.      What URL is used to perform searches of VuFind’s web index?
                                                        



