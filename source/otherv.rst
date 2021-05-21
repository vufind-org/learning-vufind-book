Other Search Backedns
*********************


Chapter 15. Other Search Backends
#################################

Learning Objectives
-------------------

After reading this chapter, you will understand:

•       How to integrate VuFind with non-Solr-based search backends.
•       Which commercial services might make useful complements to VuFind.
•       The difference between VuFind and a “web-scale discovery service.”

15.1 Introduction to Search Backends
------------------------------------

As was previously discussed in section 4.5.2, VuFind refers to the bundles of code it uses to communicate with various search systems as “search backends.” VuFind’s backends are designed to be as interoperable as possible, so they can share common templates and tools (like recommendation modules); this design provides for a more consistent user experience and also reduces the amount of programming work needed to provide flexible, customizable user interfaces for a variety of systems.

So far in this book, the examples have focused on backends that use VuFind’s local Solr index. However, VuFind also includes other backends for communicating with a variety of systems (mostly subscription-based) which can be used to provide even more options to your users. This chapter describes those alternatives in more detail.

15.2 Web-Scale Discovery Services
----------------------------------

VuFind is often referred to as a “discovery system,” since it is software that helps users discover and explore resources. However, this sometimes causes confusion, because there are a variety of commercial indexes that also refer to themselves as “discovery services,” yet they are not exactly the same thing. VuFind is a user interface that can have data added to it; “web-scale discovery services” are subscription-based indexes that provide huge, pre-built aggregated indexes along with interfaces and APIs providing search access to those indexes.

A commercial discovery system’s native interface may look a lot like VuFind, but it is not quite the same: because VuFind is an open source package that you can configure and customize on your own, it offers more flexibility and local control than a commercial discovery service. Conversely, the commercial discovery services often have access to data that individual users cannot easily obtain for indexing into their VuFind instances, and managing an index of that scale is a huge job that an individual library would be unlikely to wish to take on, so these services may have an advantage in terms of data availability.

Fortunately, it is possible to reach a “best of both worlds” compromise: if your library can afford to subscribe to a commercial discovery service, VuFind most likely has a search backend which can use that service’s API to pull search results directly into VuFind’s interface. This means that you can leverage the rich indexing work done by the commercial provider while also customizing the look and feel using VuFind’s built-in flexibility.

As of this writing, most of the major commercial discovery systems are supported by VuFind. EDS (EBSCO Discovery Service), Primo Central and Summon support are part of the core package; work in progress on WorldCat Discovery exists, and could be brought up to date and completed if sufficient interest were expressed. To use any of these services, it is simply a matter of adding some settings to the appropriate .ini file (and, in some cases, config.ini) as discussed in section 4.5.2. The service can then be combined with a local Solr index using the techniques described in chapter 13 and section 14.3. If you do not need to maintain a local index and instead wish to use one of these services as your primary search method, the “defaultModule” setting in config.ini can be used to switch VuFind’s default behavior – for example, you could change this from “Search” to “Summon” to make Summon the default home page of your VuFind instance.

15.3 Other Supported Services
-----------------------------

While web-scale discovery may offer the broadest range of additional search results in a VuFind instance, there are a number of additional supported services that may provide useful additions to your VuFind setup.

15.3.1 WorldCat
_______________

Many libraries of all sizes participate in OCLC’s services for copy cataloging and interlibrary loan. OCLC members have access to the WorldCat API, which makes it possible to search the organization’s union catalog and identify which libraries hold which books. VuFind includes a WorldCat backend which can be very useful for implementing a “books held by other libraries” area on a combined search screen. To set this up, you will need to get an API key from OCLC, and then you can fill in the [WorldCat] section of config.ini and the WorldCat.ini file to set things up.

15.3.2 BrowZine
_______________

BrowZine is a subscription service primarily designed to provide users with a convenient way to browse recent periodicals online. It also includes an API that can be useful for searching for specific journal titles held by the library. VuFind includes a backend that allows this journal title search to be incorporated into the system.

15.3.3 EIT (EBSCO Integration Toolkit)
______________________________________

In addition to their web-scale discovery service, EDS, EBSCO also provides the less feature-rich “EBSCO Integration Toolkit,” which provides API-based search access to EBSCO-hosted databases. If your library has EBSCO subscriptions but has chosen not to subscribe to a full web-scale discovery service, EIT may provide a way to expose some database results through VuFind. Note that there has been discussion in the past about eventually replacing EIT with a different service, so this option may not be available indefinitely.

15.3.4 LibGuides
________________
The popular LibGuides service, used for developing library subject guides and other similar resources, provides an API for searching available guides. VuFind is able to use this as a search backend, making it possible to highlight relevant guides in a combined search. Note that it may also be possible to index LibGuides pages into a local web index using a sitemap as described in chapter 12; however, direct API access might be preferable if it is important to you to reflect page changes in real-time, if you do not wish to maintain a web index, or if you want to highlight guides independently of other web content.

15.3.5 Pazpar2
______________
Pazpar2 is an open source federated search application which can be configured to perform a search across multiple databases and applications and then return a unified list of search results. VuFind can then be configured to fetch results from Pazpar2 through a search backend. Federated search is no longer a popular technology, due both to performance limitations and the complexity of configuring and maintaining it. Due to inherent limitations of Pazpar2’s features, the integration with VuFind lacks some functionality found in other backends (such as the ability to save records to favorites). For all of these reasons, using Pazpar2 is not recommended; however, support exists for those rare situations where it may be the best solution to a problem.



Additional Resources
---------------------

The third-party content page in VuFind’s wiki should list all of the currently-supported backends: https://vufind.org/wiki/configuration:third-party_content. To learn more about the process of developing a whole new search backend, you can refer to this wiki page: https://vufind.org/wiki/development:howtos:connecting_a_new_external_data_source.

Summary
-------

In addition to presenting content from a locally-maintained Solr index, VuFind can also provide search access to a variety of third-party systems, including “web-scale discovery.” By combining VuFind’s inherent flexibility with commercial services that provide data and functionality beyond the capabilities of your local team, you can develop a “best of both worlds” discovery experience for your users

Review Questions
----------------

1.      How do you change the default search presented by VuFind (e.g. replace Solr with Primo Central or EDS)?
2.      Which web-scale discovery services are supported by VuFind, and how can they be configured?
3.      Which search backend is most useful for listing books held by other libraries?


                        
