#######################################################
Chapter 13. Combining Different Types of Search Results
#######################################################

Learning Objectives
--------------------

After reading this chapter, you will understand:

•       How to combine different types of searches into a single interface.
•       How to activate additional search navigation features.
•       The roles of the combined.ini and searchbox.ini configuration files.


13.1 Understanding combined.ini
-------------------------------

So far in this book, two distinct types of VuFind searching have been introduced: the default bibliographic search and the web search (see chapter 12). Even more options will be discussed in chapter 15. Search engines have trained users to search using a single box, but presenting heterogeneous search results in a single list does not always make sense, and can introduce complex relevance ranking challenges. A popular solution to this problem is the “Bento Box” approach, in which a single search query reveals multiple categories of search results. VuFind supports this type of searching through the combined.ini configuration file.

The combined.ini file controls the behavior of the /Combined page of your VuFind instance (e.g. http://localhost/vufind/Combined). It allows you to search across any number of search backends and display those results on screen together in a pattern that you define.

As with all of VuFind’s configuration files, the most complete and up-to-date documentation on options and values can be found in the comments contained within the file itself. The configuration file has a handful of reserved sections (Basic_Searches, HomePage, Layout and RecommendationModules); any other sections added to the file must have names that correspond with search backends supported by VuFind (see section 4.5.2 for an introduction to search backends).

The reserved sections affect global aspects of the combined screen: how to lay out the Bento Boxes, what to display on the home screen before a search is performed, which recommendation modules to display above or below the combined results, etc. The search backend-specific sections contain settings specifying how to display results from those backends: how many results to include in the box, whether or not to load the results through AJAX, how to label everything, etc.

For example, suppose we want to show our standard Solr search results side-by-side with our website search results. We want to load the Solr results “inline” (so they are immediately available when the page loads) but we want to load the website results using AJAX (meaning that they will “pop in” shortly after the rest of the page has loaded – using AJAX makes the overall page load more quickly, though some results may take additional time to appear). The configuration would look like this ($VUFIND_LOCAL_DIR/config/vufind/combined.ini):

.. code-block:: ini

   [Layout]
   columns = 2

   [Solr]
   label = Catalog
   more_link = "More Catalog Results"

   [SolrWeb]
   label = Website
   ajax = true
   more_link = "More Website Results"


VuFind’s combined search feature also includes the ability to create specialized boxes by applying filters to search backends. For example, if you wanted to highlight online materials in your catalog, you could create a special box to show them. To do this, you simply need to add a colon and a differentiating name after the search backend name in the configuration, then populate the filter or hiddenFilter setting. The difference between a “filter” and a “hiddenFilter” is whether or not the user interface will show the filter as an applied facet. If you use “filter” and the user clicks through to the full search results, they will see the filter and can remove it; if you use “hiddenFilter,” the filter will be applied but there will be no control to get rid of it. The “hiddenFilter” control is useful in combination with filtered search tabs (described below in section 13.3), but “filter” will provide a better user experience in other situations. The “online” example could be set up like this:

.. code-block:: ini

   [Solr:online]
   label = Online Material
   hiddenFilter = "format:Online"
   more_link = "More Online Material"


This example uses a hiddenFilter because it is designed to complement a future search tab example.

Note that if you copy the default combined.ini file from $VUFIND_HOME/config/vufind/combined.ini, you will have to comment out any example sections representing search backends that you do not use; the simple presence of a backend-related section in the file will cause that backend to be included in combined results.

If you want to make this combined search the default for your entire site, you can edit your $VUFIND_LOCAL_DIR/config/vufind/config.ini file and change the “defaultModule” setting from “Search” (the default, which uses the primary Solr biblio core as the standard search) to “Combined” (which will make the global search box do a combined search by default, and will make the standard home page of VuFind the Combined home page).

13.2 Understanding searchbox.ini
--------------------------------

VuFind offers a configuration file to control the behavior of the search box that appears on every page of the application: searchbox.ini. By default, the drop-down menu next to the search box will only include options relevant to the currently-active backend. For example, if you are looking at Solr bibliographic search results, you will see the options configured in searches.ini; if you are looking at web results, you will see the options configured in website.ini. If you introduce combined searching, however, this behavior may no longer be desirable, and it may be preferable to display all of the options in the menu at all times, for a more consistent experience.

To turn on combined options in the search box, simply copy $VUFIND_HOME/config/vufind/searchbox.ini into $VUFIND_LOCAL_DIR/config/vufind/, turn on the combinedHandlers option in the [General] section, and populate the [CombinedHandlers] section with the appropriate options. For example:

.. code-block:: ini

   [General]
   combinedHandlers = true

   [CombinedHandlers]
   type[] = VuFind
   target[] = Solr
   label[] = Catalog
   group[] = false

   type[] = VuFind
   target[] = SolrWeb
   label[] = Website
   group[] = false

When editing the [CombinedHandlers] section, it is important to ensure that all of the settings end in brackets ([]) and that every block includes all of the values. Otherwise, the configuration may be interpreted incorrectly.

The “type” of “VuFind” simply indicates that these are internal VuFind search backends; you can also use a type value of “External” if you want to allow the VuFind search box to redirect into a third-party system. (When using “External,” the “target” value is a URL instead of a search backend name). The “label” setting should be self-explanatory, and the “group” setting can be used to group related options together under a heading within the drop-down menu, if you have a large number of options that need to be organized more hierarchically.

As usual, additional options (such as the ability to incorporate alphabetical browse options into the search drop-down menu) are documented through comments in the file.

13.3 Configuring search tabs in config.ini
------------------------------------------

In addition to the value of searching multiple systems at once and having access to all options through a single drop-down menu, there is one more feature which can help users navigate complex search environments: tabs. While the combined search screen provides a summary of the first page of results from multiple search backends, users will often need to click into a single result set to access deeper results and features like facet controls. Once a user has focused in on a specific result set, it is sometimes useful to have a quick way to switch into a different one. VuFind’s search tab feature offers this functionality.

When search tabs are enabled, tabs will appear near the search box. When search results are displayed, clicking on a different tab will transfer the current search terms (and, when possible, the search handler) to a different search backend. Tabs provide a quick way to switch between different detailed search result views. The transfer of search handler settings is based on name matching – for example, if you are performing an “Author” search in Solr and you click to a different tab, VuFind will look for a search handler whose description matches “Author;” if no match is found, the default handler will be applied.

Configuration of search tabs takes place in config.ini’s [SearchTabs] section. Simply create a map of search handler names to labels. To continue this chapter’s example of combined bibliographic and website searching, you could use these settings:

.. code-block:: ini

   [SearchTabs]
   Solr = Catalog
   SolrWeb = Website

VuFind’s search tab feature also includes the ability to create specialized tabs by applying filters to search backends, similar to the way combined search Bento Boxes can be filtered. To do this, you simply need to add a colon and a differentiating name after the search backend name in the configuration. Then you need to add an entry to the [SearchTabsFilters] section of the configuration specifying the filter(s) to apply. The “online” tab example from section 13.1 could be set up as a tab like this:

.. code-block:: ini

   [SearchTabs]
   Solr = Catalog
   Solr:online = Online Materials
   SolrWeb = Website

   [SearchTabsFilters]
   Solr:online = "format:Online"


Additional Resources
--------------------

Some additional information on the subjects discussed in this chapter can be found on the “Combining Search Types” page in VuFind’s wiki (https://vufind.org/wiki/configuration:combining_search_types). A video about combining searches can be found here: https://vufind.org/wiki/videos:combining_search_types.

Summary
-------

VuFind supports many different types of searching; by configuring combined.ini, searchbox.ini and config.ini correctly, it is possible to make the user’s experience more convenient and understandable while navigating the available options in your environment.

Review Questions
----------------

1.      How can you add or remove search backends on your combined search results screen?
2.      Which configuration file can be used to display options from multiple search backends in the drop-down menu next to the search box?
3.      How can you create a filtered search tab?
