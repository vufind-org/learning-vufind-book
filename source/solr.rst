Part 2. Configuration and Administration
****************************************

Chapter 5. Understanding Solr
#############################

Learning Objectives
-------------------

After reading this chapter, you will understand:

•       The purpose of Solr, and how it differs from other related tools like databases.
•       Basic Solr terminology, such as “field type” and “analyzer chain.”
•       How to interact with Solr, including some basic syntax.
•       How to configure VuFind’s use of Solr.


5.1 Solr Overview
-----------------

Apache Solr is an open source search platform, designed to support fast and flexible full text searching. Since VuFind is primarily concerned with searching, Solr is an ideal engine to drive most of its functionality. While VuFind provides reasonable default settings and tools that save users from having to work directly with Solr, an understanding of how the tool works can be helpful when troubleshooting problems and customizing search behavior. Describing Solr in detail is beyond the scope of this book, but this chapter will provide high-level overviews of some key features; see the “Additional Resources” at the end for some suggestions if you wish to learn more.

5.1.1 Indexes vs. Databases
___________________________


There are many different types of computer systems for storing and retrieving data, and different systems are better suited to different tasks. In fact, while VuFind uses Solr for searching, it also uses a relational database (usually MySQL/MariaDB or PostgreSQL) for storing persistent data like user accounts. Users who have a strong background in databases can find working with Solr confusing at first, so it is important to explain how an index like Solr differs from a relational database like MySQL.

In a relational database, data is stored in a collection of rigidly-structured tables which aim to break content down into the smallest possible pieces. Tables can define relationships with one another so that the small pieces can be recomposed into more complex wholes through querying. This allows data to be stored with very little duplication, which makes maintenance easier. The enforcement of rules helps to prevent certain types of common data entry errors. The clear expression of data types and relationships makes it possible to query the data in complex ways. A relational database is an excellent tool for managing highly structured data. However, it may not be as useful when dealing with extremely heterogeneous information, and searching across many fields and many tables can be slow due to the work involved in fitting all the pieces back together again.

An index like Solr differs from a relational database in many significant ways. While it imposes some structure on data, that structure is much less complex: a collection of documents containing named fields. There are fewer rules, and no safeguards to enforce data integrity, because data integrity is not a major concern of an index. Its job is simply to accept queries and find documents that match those queries as quickly and accurately as possible. To meet this goal, it makes a useful trade-off between resource consumption and performance, using significant amounts of memory and disk space to create data structures that help it to very rapidly identify matches. Solr is very good at searching, but it is most powerful when it is used to add search capabilities to data that is created and managed in some other system; while Solr can be used for end-to-end data management, doing so can be problematic, especially when upgrading to a new version of Solr or making changes to the structure of your documents.


In the real world, distinctions between relational databases and indexes can sometimes be blurry; most major relational databases have added full text searching capabilities that resemble Solr, and over time, Solr has made it more feasible (though still not necessarily desirable) to manage data directly in the index. However, even with these caveats, it remains true that each type of system has strengths and weaknesses, and VuFind has been designed to play to the strengths of its components. This is why VuFind only uses Solr for searching, and other tools are used for managing persistent data.

5.1.2 Solr’s Data Model
_______________________

As noted earlier, the key idea of Solr is that it builds an index of documents, each of which is split into named fields (like title, author, subject, etc.). The fields allowed in each document are defined by a configuration file, referred to as a schema (see, for example, $VUFIND_HOME/solr/vufind/biblio/conf/schema.xml), which specifies the type of data each field may contain, and the rules for processing each data type.

5.1.3 Analyzer Chains
_____________________

A lot of Solr’s power comes from the ability to define processing rules, or “analyzer chains,” for different data types. There are several reasons why the ability to process different types of data with different rules is important. Just as in a spreadsheet program, dates, numeric values, and text values need to be treated differently from one another when being used for sorting or filtering. For text fields, different degrees of precision may be desired for matching fields, depending on whether they are intended for keyword searching or faceting. Sometimes, special kinds of text require special rules; for example, if your documents contain the same information in different languages.

Solr uses its analyzer chains at two points in time:
•  When documents are added to its index (for example, with the use of SolrMarc, as described in chapter 3), all of their content is analyzed, and the results are saved in the index.
•  When users perform searches by submitting queries, the query content is also analyzed, and the results are compared against the index to find matches.

Because the same rules can be applied to both the search being performed and the contents being searched, it is possible to clean up minor irregularities and increase the probability of finding a useful match.

An example should make this more clear. Imagine that we have a title field designed for full text searching. We want to allow users to match individual keywords within a title rather than having to type the whole thing, so we start our analyzer chain with a “tokenizer” to break text apart into individual words. We don’t want to force the user to match the case of the original text, so we apply a lower-case filter to normalize all text to lower-case. We want to treat different forms of the same word (e.g. work/works/working) as the same word for matching purposes, so we apply a type of filter called a stemmer to simplify words with common suffixes.

Now suppose we index a book with the title “Working with Solr.” As Solr loads the data into its index, it will apply the rules we defined.

Step 1: Tokenize – “Working with Solr” will become a set of tokens, broken apart on whitespace:
Working, with and Solr.

Step 2: Lower-case filter – Working, with and Solr will now become working, with and solr.

Step 3: Stemmer – Our hypothetical stemmer will remove common suffixes, so “working” will become “work” and our tokens now become work, with and solr. These values are stored in the index.

Now, a user comes along with their Caps Lock turned on, and performs the search “SOLR” against our index. The analyzer chain is applied again, but it won’t have much to do; there’s only one token, and no stemming is necessary, but the lower-case rule will change “SOLR” to “solr,” and then the index will find that this token matches one of the tokens found in the title field of the “Working with Solr” document.

Sometimes, the side effects of analyzer chains can be unintuitive. For example, you should be able to see how a search for “SOLR WORKS” would reduce down to solr and work and thus match the tokenized results of “Working with Solr” on two tokens. Some users might not think this is a good match. It will sometimes be up to you to determine why a given query matches a given result, and to make decisions about whether configurations need to be adjusted to change the rules.

Of course, this example is an extreme simplification of how Solr works, but the point is this: Solr applies rules to reduce inputs into tokens, and then it provides search results by matching tokens. This core idea is quite simple; the power and flexibility in the system comes from the ability to define an infinite variety of rules controlling how inputs turn into tokens.

5.1.4 Relevance Ranking
_______________________

Another powerful aspect of Solr is the ability to rank documents by relevance in search results. When performing a query, it is possible to provide rules for relevance-ranking the results; for example, when performing an “all fields” search, you might decide that a token match in a title field is more important than a token match in an author field. When searching very large collections of documents, the ability to adjust relevance to suit your audience and documents can make the difference between success and failure in searching. Users rarely navigate past the first screen of search results, so getting the best results to the top of the list is very important. Relevance ranking is discussed in more detail in the next section.

5.2 Customizing Search Behavior with searchspecs.yaml
-----------------------------------------------------

As described in section 4.3, the searches.ini file gives you control over which search options are presented to the user, and the order in which they are presented. However, a different configuration file, searchspecs.yaml, gives you fine-grained control over how these search options actually behave: which Solr fields they search, how they determine relevance, and what special parameters they send along to Solr.

5.2.1 Working with YAML
_______________________

As the filename suggests, searchspecs.yaml is a YAML file. YAML is a format designed for storing complex data in a human-readable format. When search configuration functionality was added to VuFind, the .ini format used for most of VuFind’s other configuration files was too simplistic to easily represent the complexity of the available options. Of other commonly-used file formats, XML seemed too verbose, and JSON did not support the ability to insert human-readable comments. Thus, YAML was chosen as a best-of-all-worlds solution.

YAML does have some drawbacks, most particularly a sensitivity to misplaced whitespace. If you edit the searchspecs.yaml file, be very careful that you follow the existing indentation and alignment conventions, since a line out of place can cause the file to be misinterpreted. Fortunately, many of the changes you may wish to make are simply a matter of changing numbers and following existing examples, so it should be possible to avoid pitfalls with a bit of caution.

5.2.2 Anatomy of searchspecs.yaml
_________________________________

The searchspecs.yaml file (found in $VUFIND_HOME/config/vufind along with VuFind’s other configuration files) begins with a long block of comments summarizing the available settings and options; this comment block should always be treated as the most up-to-date documentation on the behavior of the file.

Under the documentation comment, there are a number of sections defining VuFind’s “search handlers” – the options that can be configured in searches.ini, which define different types of searches (title, author, subject, etc.) that users can perform. Here is an example, defining VuFind’s “Author” search:

.. code-block:: console

   Author:
     DismaxFields:
        - author^100
        - author_fuller^50
        - author2
        - author2_fuller
        - author_additional
        - author_corporate
        - author_variant
        - author2_variant
     DismaxHandler: edismax

The top-level “Author:” begins the block. Everything beneath that is indented by two spaces to show that it is contained within the section – it is this type of indentation that must be carefully maintained to ensure the file is read correctly by the software. Within the block, there are two main settings: “DismaxFields,” which controls relevance ranking (see 5.2.4 below), and DismaxHandler, which in this instance tells VuFind to use Solr’s “Extended DisMax” search functionality. Quite a few other options are supported, some of which will be described in more detail below, and the rest of which can be found described in the aforementioned documentation comment embedded in the searchspecs.yaml file.

5.2.3 Lucene, DisMax and Extended DisMax
________________________________________


You will see the word “DisMax” a lot when reading the file, so it is worth taking a moment to explain it. “DisMax” is short for “Disjunction Max,” which is a powerful Solr query mode which allows the same term to be searched across a number of different fields, with relevance ranking rules applied to bring the “best” matches to the top of the list (see https://cwiki.apache.org/confluence/display/SOLR/DisMax for more details).

Before “DisMax” was introduced, VuFind relied on a much more complicated (and less accurate/reliable) method of generating search queries using the Boolean-based syntax of Solr’s underlying index engine, Lucene. For many years, VuFind operated in a hybrid mode, using DisMax for most searches but switching over to the old method for queries that used features unsupported by DisMax. This old method is still used in VuFind for some very specialized types of searches (see 5.2.6 below for more details), but since the introduction of Solr’s “Extended DisMax” mode, which allows a “best of both worlds” approach combining both DisMax behavior and traditional Boolean logic, VuFind’s search configurations have been significantly streamlined and simplified. Understanding the differences between all of these different search methods is not critical to making the most of VuFind, but knowing a little of this history may make some of the documentation easier to understand.

5.2.4 Adjusting Relevance
_________________________

Whenever VuFind performs a DisMax search, it looks for user search terms across a set of fields, then does some mathematical analysis on the matches it finds to calculate which documents have the best matches, for the purpose of putting the result set in a useful order. This relevance calculation is informed to a large extent by relative weights applied to the fields being searched. In searchspecs.yaml, the DismaxFields section serves the dual purposes of defining which fields Solr should search across and assigning relative weights to those fields. To return to the Author example from above:

.. code-block:: console

   DismaxFields:
       - author^100
       - author_fuller^50
       - author2
       - author2_fuller
       - author_additional
       - author_corporate
       - author_variant
       - author2_variant

This setting tells Solr to search across eight fields: author, author_fuller, author2, author2_fuller, author_additional, author_corporate, author_variant and author2_variant. Note that both author and author_fuller are followed by a caret and a number: author^100 and author_fuller^50. This is how relevance is adjusted. In this rule, hits in author_fuller get 50 times more relevance weight than all of the other fields except for author (which bears twice as much weight as author_fuller). The idea of this default configuration is that when a user searches for an author, works listing that person as a primary author are more likely to be of interest than works listing that person as a secondary or additional author. Of course, your local preferences might be different; for example, perhaps your audience is more interested in corporate authors. In that case, the numbers in searchspecs.yaml can be adjusted to change the search behavior.

As with other configuration files, it is strongly recommended that you modify a copy in $VUFIND_LOCAL_DIR/config/vufind rather than editing the default version in the core. Additionally, if your local copy of the file excludes any top-level sections, these will be loaded in from the core version, so you can choose to keep only those sections that you add or change in your local searchspecs.yaml, and inherit the other default values from the core version. For example, if you only wanted to change the Author section, you could create a local searchspecs.yaml containing only “Author:” and the contents indented beneath it, and all of the other search options (Title, Subject, etc.) would continue to work based on the core defaults.

When adjusting relevance numbers, it is helpful to keep in mind that the numbers themselves don’t really mean very much; it’s their relative size to one another that matters the most. Is one field twice as important as another one? Ten times more important? A hundred times more important? It may take some trial and error to find the right balance that yields the most useful results. Sometimes compromises will have to be made. When tuning relevance ranking, it is wise to keep notes about the reasons for your changes, and some important search queries that informed decision-making. That way, in the future, as more changes are made, you can repeat old searches and find out how they are impacted.

5.2.5 Passing Extra Parameters
______________________________

As you will see in section 5.3, Solr accepts a wide variety of parameters that impact search behavior. There is a generic section in searchspecs.yaml called DismaxParams that can be used to add arbitrary parameters to all searches performed using a particular search handler. There are a couple of common reasons you might want to take advantage of this:

5.2.5.1 Relevance Boosting
__________________________

Sometimes, when calculating relevance, ranking documents on which fields match the query is not enough. Perhaps you want to emphasize documents of a particular format. Perhaps you would like to boost more recently-published documents. Fortunately, Solr provides mechanisms to support both of these types of scenarios.

Boost queries are the simplest to use. You simply provide a bq parameter whose value is a Solr search query. Any documents that match the boost query will receive a relevance boost. For example, if you wanted to boost documents with a format field value of “Book,” you could add this configuration to the relevant searchspecs.yaml section:

.. code-block:: console

   DismaxParams
   - [bq, format:Book]

Another simple option is phrase boosting; if you specify a field name in the pf parameter, then Solr will apply extra weight to documents in which the user’s search terms appear as a phrase within the specified field. This helps to break ties between documents that contain matching terms that are far apart, and documents that contain the same matching terms closer together. This can be especially valuable for title searching. For example, you might add this to the “Title” section:

.. code-block:: console

    DismaxParams:
        - [pf, title_full]

Finally, an advanced option is to provide a boost function using the bf parameter. Solr provides several mathematical functions which can be applied to data in the Solr index and used to adjust relevance rankings. This is useful for the “boost recently published documents” scenario, as well as for situations where you have access to a numeric score that you can index directly into Solr. You will probably want to gain a deeper familiarity with Solr before attempting to work with boost functions, but when you are ready, you will find a whole chapter about them in Solr’s reference manual.

5.2.5.2 Minimum Matching
^^^^^^^^^^^^^^^^^^^^^^^^
VuFind is configured that, by default, when a user enters multiple search terms to perform a DisMax search, only records that match ALL of those terms will be returned. However, DisMax can be more tolerant of partial matching – for example, if you want to return records that match 75% of search terms, even if no records match 100% of terms. This extra level of tolerance can be useful, especially when dealing with very long queries. Solr’s mm (“minimum should match”) parameter controls this behavior, and it supports a variety of different scenarios. You can set minimum matching requirements, or you can allow tolerance for certain levels of unmatched clauses. You can even specify different rules for different numbers of incoming terms. See the Solr documentation for a full explanation of how these settings work. Here is a simple example, ensuring that at least 75% of search terms must match:

.. code-block:: console

     DismaxParams:
         - [mm, 75%]

5.2.6 Munging
______________


“Munging” is a slang term for data manipulation, often implying a somewhat crude or rudimentary approach. When working with Solr, it is usually a good idea to let Solr do most of the data manipulation. However, there are rare occasions where it may be useful to have VuFind manipulate user input before submitting it to Solr. For these situations, the searchspecs.yaml “munging” system exists.

This is probably best explained with another example:

.. code-block:: console

     oclc_num:
       CustomMunge:
         oclc_num:
              - [preg_replace, "/[^0-9]/", ""]
              # trim leading zeroes:
              - [preg_replace, "/^0*/", ""]
     QueryFields:
        oclc_num:
         - [oclc_num, ~]

This is a handler definition for searching by OCLC number, a commonly used unique identifier. OCLC numbers sometimes have leading prefixes and/or extra zeroes. This section is designed to reduce an OCLC number to its core numeric value, eliminating any prefix or leading zeroes. While it would be possible to accomplish this inside Solr by creating a custom data type in the schema, in this instance, doing outside manipulation was deemed a lighter-weight solution.

As you can see, there is no DismaxFields or DismaxHandler setting in this section; this handler is going to use the older Lucene query syntax for searching. It doesn’t need DisMax because it is only going to search a single field.

The CustomMunge section defines a text processing rule called “oclc_num” which consists of two steps: first, eliminating all non-numeric characters, and next, trimming off any leading zeroes. Both of these steps are accomplished using regular expressions. See the comments at the top of searchspecs.yaml for a list of valid munging functions. If you are not familiar with regular expressions, many tutorials and primers on the subject can be found online.

The QueryFields section specifies which field or fields will be searched, and what rules to apply to each field. In this case, we are searching only one field (oclc_num), and we are applying only one rule (also named oclc_num in this example). To be clear: the “oclc_num” preceding a colon on a line by itself refers to the field being searched; the “oclc_num” inside brackets refers to the munge rule. The tilde (~) following the munge rule indicates that no extra relevance boost needs to be applied; in situations where multiple rules are being applied, it is possible to rank them relative to one another.

Fortunately, because DisMax provides a much simpler configuration and works for the majority of cases, it is rare that users need to work with or understand these older munge-based search types; however, a basic understanding of how to read them may be helpful, especially if you are troubleshooting a search that uses them. Call Number searching is probably the most common remaining use case for this type of search configuration.

5.3 Troubleshooting Solr with VuFind’s Debug Mode
-------------------------------------------------

As mentioned earlier, VuFind does the hard work of interacting with Solr for you, and exposes most of the options you will need through configuration files. When you perform a search in VuFind, it translates your query into a Solr query according to the rules defined in its configuration files, then uses that query to retrieve search results, and finally formats those results into the web page that you end up seeing.

Sometimes, if VuFind does not seem to be finding the results that you think it should, it may be helpful to see exactly what is happening behind the scenes. Fortunately, VuFind includes a debug mode which allows you to see more information about what it is doing. To turn this on, simply edit your local config.ini file, look for the “debug = false” line near the top, and change it to “debug = true.”

When debug mode is turned on, you will see information boxes scattered around VuFind’s interface full of detailed technical information. When you perform a search, some of these boxes will include full Solr query URLs. Note that, because of the way VuFind’s spell checking feature works, a single search can actually trigger multiple queries against Solr; generally speaking, when looking at VuFind debug output, the first Solr URL is the most important one.

For example, try turning on debug mode and then searching for “SOLR” as described in the example in section 5.1. You should see a debug message similar to this:


.. code-block:: console

   2020-02-17T10:28:29-05:00 DEBUG: VuFindSearch\Backend\Solr\Connector: => GET http://localhost:8983/solr/biblio/select?fl=%2A%2Cscore&spellcheck=true&facet=true&facet.limit=30&facet.field=topic_facet&facet.field=institution&facet.field=building&facet.field=format&facet.field=callnumber-first&facet.field=author_facet&facet.field=language&facet.field=genre_facet&facet.field=era_facet&facet.field=geographic_facet&facet.field=publishDate&facet.sort=count&facet.mincount=1&fq=format%3A%22Book%22&sort=score+desc&facet.pivot=callnumber-first%2Ctopic_facet&hl=true&hl.simple.pre=%7B%7B%7B%7BSTART_HILITE%7D%7D%7D%7D&hl.simple.post=%7B%7B%7B%7BEND_HILITE%7D%7D%7D%7D&spellcheck.dictionary=default&wt=json&json.nl=arrarr&rows=20&start=0&spellcheck.q=SOLR&qf=title_short%5E750+title_full_unstemmed%5E600+title_full%5E400+title%5E500+title_alt%5E200+title_new%5E100+series%5E50+series2%5E30+author%5E300+author_fuller%5E150+contents%5E10+topic_unstemmed%5E550+topic%5E500+geographic%5E300+genre%5E300+allfields_unstemmed%5E10+fulltext_unstemmed%5E10+allfields+fulltext+description+isbn+issn+long_lat_display&qt=edismax&mm=0%25&hl.fl=title_short%2Ctitle_full_unstemmed%2Ctitle_full%2Ctitle%2Ctitle_alt%2Ctitle_new%2Cseries%2Cseries2%2Cauthor%2Cauthor_fuller%2Ccontents%2Ctopic_unstemmed%2Ctopic%2Cgeographic%2Cgenre%2Callfields_unstemmed%2Cfulltext_unstemmed%2Callfields%2Cfulltext%2Cdescription%2Cisbn%2Cissn%2Clong_lat_display&q=SOLR

This is a fairly intimidating block of text, but if you break it apart into chunks, you will see that it is just a long list of parameters being passed to Solr. If you split the URL apart around the ampersand (&) characters, you will see many of the individual settings are fairly straightforward. Some examples:

*q=SOLR*                          Set the query to “SOLR”
*rows=20*                         Return twenty results
*facet.field=topic_facet*         Include values from the “topic_facet” field as one of the facet options
*sort=score+desc*                 Sort by relevance score in descending order


All of these parameters (and many others) are documented in the online Solr documentation (https://lucene.apache.org/solr/) and clarification can usually be found with the help of the search engine of your choice.

You should be able to copy the URL from this message directly into your web browser to see the Solr response, as well as a more readable summary of the input parameters. (Note that, if the URL starts with http://localhost, you may have to replace “localhost” with the name of the Solr/VuFind server if you are trying to review the results from a different machine – and, of course, access from another machine will only work if firewalls are set up to allow it. For security reasons, cross-machine access to Solr should generally be restricted except when needed for development or troubleshooting).

In any case, once you have access to Solr query results, you can see exactly what details are coming out of Solr, and you can edit the parameters in the URL to try different searches, sorts, etc. You can add a &debugQuery=true parameter on the end of the URL to activate a debug mode that adds an extra section to the Solr response that provides a breakdown of how Solr is processing text and how long each step of the analyzer chain is taking, which can be useful for identifying performance problems.

Figure: Web browser displaying Solr response including debugQuery details

You will also find that if you access just the base part of the URL (which will usually be something like  http://localhost:8983/solr), you will find a useful Solr administration panel which lets you explore some of the features of the platform; this also includes a helpful form for building your own queries. If you decide to learn more about Solr using the “Additional Resources” below, you will likely spend a lot of time in this interface.

Note that turning on debug mode can prevent some features of VuFind from working correctly, so it is not a good idea to leave it turned on all the time; it is just intended as a quick-and-easy way to access some technical details like Solr queries. For some more robust alternatives, including logging to files, see the troubleshooting page in VuFind’s wiki (https://vufind.org/wiki/development:troubleshooting).


Additional Resources
--------------------

There are several book-length introductions to Solr currently in print as of this writing, many published by Packt Publishing. For a shorter introduction, the official “Solr Quick Start” tutorial at https://lucene.apache.org/solr/guide/solr-tutorial.html provides a useful place to begin. For reference, the full Solr documentation can be found at https://lucene.apache.org/solr/resources.html#documentation.


Summary
-------

Solr is an index engine optimized for searching, which is why it serves as the foundation for VuFind’s default search functionality. It provides access to documents by providing field-based searching. A highly configurable search schema and a wealth of search parameters give the administrator a great deal of control over how searches are performed; VuFind exposes much of this functionality through its own configuration files, and its debug mode can help you troubleshoot problems. With a basic understandingof how Solr works and where VuFind’s configuration options can be changed, it should be possible to tune VuFind to meet the needs of local user communities.

Review Questions
----------------
1.  Why does VuFind use both a relational database and an index? What is the role of each?
2.  What is the benefit of applying the same analyzer chain to indexed text and user search queries?
3.  Name three parameters that VuFind passes to Solr when performing searches, and what their purposes are.

