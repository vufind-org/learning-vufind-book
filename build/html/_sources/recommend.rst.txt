##################################
Chapter 14. Recommendation Modules
##################################

Learning Objectives
-------------------

After reading this chapter, you will understand:

•       What a recommendation module is.
•       How recommendation module configuration works.
•       Some common use cases for recommendation modules.


14.1 What is a Recommendation Module?
--------------------------------------

Generally, when a user sees a list of search results, it is beneficial to provide them with some additional information beyond the results themselves: facet lists for limiting results, suggestions about spelling corrections or relevant alternate resources, etc. In VuFind, all of this supplemental information is displayed to the user using “recommendation modules.”

A recommendation module is a VuFind plug-in, consisting of a PHP class and an associated template, which can examine a user’s search query and/or the search results, then display useful information based on some or all of that input. Recommendation module output can be configured to display above or beside the search results, and different modules can be configured for each search handler. It is also possible to configure VuFind to display different recommendation modules when the user’s search results are empty, since a different set of suggestions may be relevant in that scenario.

VuFind includes a large variety of recommendation modules and a default configuration that makes use of those that are most commonly needed. VuFind’s built-in extensibility also makes it straightforward for developers to build their own custom modules as needed.

14.2 Understanding Recommendation Configuration
-----------------------------------------------

Each search backend (Solr, EDS, Summon, etc.) has its own separate recommendation configuration, since each system has unique characteristics and requirements; by default, all backends are set up with similar-looking configurations.

Recommendation configuration can be found in the primary .ini file for the relevant backend – e.g. searches.ini for the primary Solr search, website.ini for the Solr website search, EDS.ini for EBSCO Discovery Service, Summon.ini for Summon, etc. The most detailed notes and comments (including a list of all available modules) can be found in searches.ini, since this is the most commonly-used configuration. Configuration works exactly the same way across all backends, though some modules may only apply to specific backends. The comments will note this when relevant.

Recommendation settings are spread across several sections of the .ini files. In the [General] section, the default_top_recommend[], default_side_recommend[] and default_noresults_recommend[] settings allow defaults to be configured. These are the modules that will be loaded in the specified context(s) when more specific configurations are not found in the other relevant sections: [SideRecommendations], [TopRecommendations] and [NoResultsRecommendations]. In these latter three sections, the configuration keys are the names of search handlers, and the values are the recommendation modules that should be displayed when those search handlers are used. A special [AuthorModuleRecommendations] section also exists in searches.ini for configuration of the recommendations presented on the special author screen reached by clicking on author names in Solr search results.

This configuration setup offers a balance between ease of use and specificity. You can define default values that will be used in most situations, but you can override them with more specific recommendations as needed.

At each level of the configuration, recommendation modules are configured by providing the name of the module, optionally followed by a set of parameters separated by colons. The meanings of these parameters are detailed in the comments in searches.ini (and some examples will be explained below).

Here are the most relevant configurations from searches.ini as of VuFind 7.0:

.. code-block:: console

   [General]
   default_top_recommend[] = TopFacets:ResultsTop
   default_top_recommend[] = SpellingSuggestions
   default_side_recommend[] = SideFacets:Results:CheckboxFacets
   default_noresults_recommend[] = SwitchType
   default_noresults_recommend[] = SwitchQuery:::fuzzy
   default_noresults_recommend[] = SpellingSuggestions
   default_noresults_recommend[] = RemoveFilters

   [SideRecommendations]

   [TopRecommendations]
   Author[]            = AuthorFacets
   Author[]            = SpellingSuggestions
   CallNumber[]        = "TopFacets:ResultsTop"    ; disable spelling in this context

   [NoResultsRecommendations]
   CallNumber[] = SwitchQuery::wildcard:truncatechar
   CallNumber[] = RemoveFilters


This configuration reveals that most searches will use defaults, but there are some special cases for author searches and call number searches.


On a normal search that returns results, facet options will be displayed in two places: some above the search results (based on the “TopFacets:ResultsTop” setting, which loads facets based on the ResultsTop section of facets.ini) and some in a sidebar (based on the “SideFacets:Results:CheckboxFacets” setting, which loads facets based on the Results and CheckboxFacet sections of facets.ini). Additionally, spelling suggestions will be displayed above the results, based on the “SpellingSuggestions” setting.


On a search that returns no results, there is no need to display facets, since there are no values available. However, spelling suggestions will still be displayed, along with three recommendation modules that suggest different ways the user could adjust their search to potentially bring in more results: SwitchType (which may suggest using a different search handler), SwitchQuery (which may suggest broadening a search query with a wildcard or by removing quotes), and RemoveFilters (which may suggest turning off applied filters).


The Author search uses a special recommendation module for browsing author names in place of the usual TopFacets display. Note that the SpellingSuggestions setting is repeated here because spell checking is still desired in this context, and if you override any recommendation modules for a search handler, none of the existing defaults will be applied. By contrast, the CallNumber-specific configuration maintains the default TopFacets display but turns off spell checking, since offering spelling suggestions for call numbers can lead to confusing results. CallNumber settings for empty search results are also customized to provide more context-appropriate suggestions.


Note that for all of the repeating settings (like default_top_recommend[]), it is important to include the brackets at the end of the key to ensure that all settings are respected, and the order of the settings in the .ini file will control the order in which the recommendation modules are displayed on screen.

14.3 Example: Cross-Linking Search Types through the Sidebar
------------------------------------------------------------

Chapter 13 discussed an example of combining standard search results with website search results in a variety of ways. With recommendation modules, it is possible to add another layer of combined searching: you can display results from one backend as a sidebar in another result set. For example, you could show the top five web results in a sidebar next to main catalog results, and vice versa.

VuFind provides two recommendation modules that can be used to meet this need: CatalogResults, which displays results from the main Solr biblio core, and WebResults, which displays results from the Solr website core. Both of these modules accept two parameters: the name of the URL parameter containing search terms (which defaults to “lookfor,” which does not need to be changed for this example) and the number of search results to display in the sidebar (which defaults to 5). We just need to turn on WebResults in searches.ini and CatalogResults in website.ini.

This can be set up by following these steps:

1.      Copy searches.ini  and website.ini from $VUFIND_HOME/config/vufind/ to $VUFIND_LOCAL_DIR/config/vufind/ if you have not previously customized these files.

2.      Edit $VUFIND_LOCAL_DIR/config/vufind/searches.ini, and add *default_side_recommend[] = WebResults* to the [General] section of the file. In a default VuFind configuration, there should be no customizations in the [SideRecommendations] section, but if you have made customizations there, you will want to add WebResults to each of the customized search handlers as well.

3.      Edit $VUFIND_LOCAL_DIR/config/vufind/website.ini and add *default_side_recommend[] = CatalogResults* to the [General] section if it is not already there (in recent VuFind releases, this is already turned on by default).

Now VuFind should display brief previews of web results in standard result listings and vice versa.

If you wanted to display a different number of results in the recommendation boxes (for the sake of example, 3), you could edit the configuration lines to read *default_side_recommend[] = WebResults::3* and *default_side_recommend[] = CatalogResults::3*. The double colon is present because we are leaving the first parameter blank.

If you want to customize the look and feel of the recommendation boxes, each recommendation module has its own template which you can easily override in a local theme, as discussed in chapter 7 (particularly section 7.4). The naming convention for these template files is $VUFIND_HOME/themes/[theme name]/templates/Recommend/[module name].phtml – so, for example, if you wanted to override the WebResults display as defined in the bootstrap3 theme, and your local custom theme was named localtheme, you could copy $VUFIND_HOME/themes/bootstrap3/templates/Recommend/WebResults.phtml to $VUFIND_HOME/themes/localtheme/templates/Recommend/WebResults.phtml, and then make edits to the latter file.

14.4 Example: Displaying Extra Links for Empty Search Results
--------------------------------------------------------------

It is often useful to provide links to specific resources related to a search. For example, you might have a “search tips” page on your website which could provide guidance for users having difficulty with searches. VuFind includes a recommendation module named RecommendLinks which can render such a list. The RecommendLinks helper takes two parameters: the name of an ini file, and the name of a section within that file; it uses these to locate the list of recommendations to display. If no extra details are specified, it will look in the [RecommendLinks] section of searches.ini. Allowing configuration of the location of links means that the same RecommendLinks module can be used in different contexts to display different lists of links.

To implement the example of a link to a “search tips” guide when a user performs a search with no results, we could simply add *default_noresults_recommend[] = RecommendLinks* to the [General] section of $VUFIND_LOCAL_DIR/config/vufind/searches.ini, and then, in the same file, add this to the [RecommendLinks] section:

.. code-block:: console

   Search Tips = http://library.myuniversity.edu/search-tips


(where “Search Tips” is the link text that will be displayed to the user, and http://library.myuniversity.edu/search-tips is the desired link URL).

As in the previous example, the presentation of the links can be customized by overriding the Recommend/RecommendLinks.phtml template in a custom theme.

Additional Resources
---------------------

VuFind’s recommendation module wiki page can be found at https://vufind.org/wiki/development:plugins:recommendation_modules.

Summary
-------
Recommendation modules are used by VuFind to display supplemental information that complements search results. They are highly configurable, so you can use them to communicate important information specific to certain search backends and/or search handlers.

Review Questions
----------------

1.      Where can you find a complete list of recommendation modules, including parameters?
2.      Why are the brackets ([]) important at the end of settings like “default_top_recommend[]”?
3.      If you wanted to add a new recommendation module only shown for Subject searches, while also displaying the existing default modules, how would you configure that?
