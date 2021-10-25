###################################
Chapter 9. Customizing Record Views
###################################

Learning Objectives
-------------------

•       VuFind®’s record model, and the purpose of record drivers and the record data formatter.
•       How to customize the display of existing fields.
•       How to add new fields to records.

9.1 Understanding Record Drivers
--------------------------------

VuFind® is designed to allow searching and display of a wide variety of record types across a wide variety of different systems. Its design strikes a balance between taking advantage of common features, so that the same code can be applied to multiple formats and systems, and allowing differences to be recognized, so that distinct individual characteristics of particular records can be represented and taken advantage of.

A key component of this design is the concept of “record drivers.” A record driver is a bundle of PHP code and templates representing a particular metadata format. This code is responsible for translating the specific details of that format into more general concepts that VuFind® is familiar with, and the templates are responsible for specifying exactly how VuFind® should display records of that type in various areas of its user interface: search results, record views, favorite lists, etc. All of VuFind®’s other code interacts with record drivers rather than directly with specific types of records.

Because all of the format-specific logic resides inside the record drivers, and because all record drivers expose the same basic functionality, VuFind® is able to use the same code to present data from a wide range of systems. Because each record driver can provide its own templates, it is possible to display fields and express concepts on a format-by-format basis even if the broader VuFind® system has no higher-level awareness of them. These two characteristics (code that conforms to a “least common denominator” and templates that allow expression of very specific format-specific features) contribute significant power and flexibility to VuFind®.

If every record driver had to define all of its own functionality and templates, it would be a lot of work to create new drivers, and the code would contain a lot of duplication. Fortunately, similar to themes, record drivers use inheritance to share functionality, so each driver has a parent driver and then specifies the ways in which it differs from that parent.

Every record driver in VuFind® inherits from a top-level class called AbstractBase which defines some of the most low-level, fundamental functionality shared by all record drivers. This is extended by a driver called DefaultRecord, which defines some more specific functionality related to bibliographic records. This in turn is extended by drivers that add more details related to specific platforms like Solr, WorldCat, etc. VuFind® has some special behavior related to Solr-based record drivers: when it retrieves a record from Solr, it looks at the record_format field and uses its value to determine a record driver name. For example, if record_format is set to “marc”, it will load the SolrMarc record driver. If it can’t find a driver matching the format value, it will fall back to using “SolrDefault,” which is the parent of most of the other Solr-based drivers. This allows a single Solr index to contain a variety of data formats while allowing VuFind® to handle each of those formats differently if necessary.

Inside your theme’s templates directory (assuming you are looking at a theme that contains templates), there is a RecordDrivers directory. This contains subdirectories that correspond to the names of record drivers. When VuFind® needs to display record-related content, it will search this directory for a template matching the relevant record driver. If no match is found, the parent record driver’s directory will be searched, and so on. In the default VuFind® themes, the majority of templates are found in the DefaultRecord driver’s folder, with only a few special overrides being defined for more specific drivers. When record driver templates are rendered, VuFind® automatically sets them up with a property called $this->driver, which points to the actual record driver object, making all of its data readily available for display.

Many VuFind® users never need to create their own record drivers, and many customizations can be accomplished by modifying or extending existing drivers. However, understanding the role of record drivers in VuFind®’s design will make it easier to understand the platform as a whole. Additionally, VuFind® users with more complex use cases (like an index of records representing substantially different metadata formats or material types) can leverage this system to create a powerful and specialized user experience.

9.2 Understanding the Record Data Formatter
-------------------------------------------

Prior to release 4.0, many of VuFind®’s record driver templates were long and repetitive, building large tables of data extracted from metadata records and Solr. This had the advantage of simplicity, but it was somewhat difficult to read, maintain and customize. It was hard to quickly identify which fields were being displayed in a particular template, and making local customizations required duplicating large amounts of template code, which in turn led to laborious reconciliation work when core templates changed during upgrades.

To address these issues, VuFind® 4.0 introduced a new view helper called the RecordDataFormatter, whose job is to extract a collection of data from a record driver following a configurable set of rules. This change does make VuFind®’s learning curve a bit steeper, since making the most of the RecordDataFormatter’s configuration requires some understanding of PHP, and there are quite a few configuration options to navigate. However, the learning cost is counterbalanced by a solution which resolves the template length, readability and maintenance challenges of the old approach. Additionally, for users who do not like the RecordDataFormatter, there is no mandate to use it; it is a view helper used in VuFind®’s default templates, but there is nothing preventing you from building a custom template using the older, simpler approach if you prefer having total control over how things are retrieved and laid out.

To see how the RecordDataFormatter is configured, you can look at the factory code which builds the view helper, which is found in $VUFIND_HOME/module/VuFind/src/VuFind/View/Helper/Root/RecordDataFormatterFactory.php. If you are not familiar with PHP, this file will be quite intimidating at first glance, but once you recognize some basic patterns, most of it should make sense even if you do not fully understand the programming language behind it. At the top of the file, there is a method named __invoke(), which does the high-level
work of building the view helper. You will notice several “setDefaults” lines, which establish default rules for which fields should be retrieved on particular templates. For example:

.. code-block:: php

   $helper->setDefaults('core', [$this, 'getDefaultCoreSpecs']);

Indicates that when rendering the core.phtml record driver template, the RecordDataFormatter should use the configuration defined in the *getDefaultCoreSpecs()* method.

If you scroll down through the file, you will eventually find the definition of the getDefaultCoreSpecs method, which begins with *function getDefaultCoreSpecs()*. You will see that this method, like all of the others written to establish default configurations, makes use of a utility called *RecordDataFormatter\SpecBuilder*, which is a PHP class containing code which helps build up the configuration array expected by the RecordDataFormatter. You don’t have to use the SpecBuilder, but it makes the configuration code more concise, and it will also help you extend and rearrange the configuration if you wish to make customizations. An example of this will be shown below in section 9.3.

The SpecBuilder has several methods, including *setLine, setMultiLine*, and *setTemplateLine*. Each of these are shorthand for different types of configuration that the RecordDataFormatter can understand, and each sets up the display of some of the data in the final output. The difference is simply how the details are retrieved and formatted.

The *setLine* method offers the absolute simplest example. For example, take a look at this line from *getDefaultDescriptionSpecs()*:

.. code-block:: php

   getDefaultDescriptionSpecs():
   $spec->setLine('Physical Description', 'getPhysicalDescriptions');

This simply says “retrieve any values from the record driver’s *getPhysicalDescriptions()* method, and display them with a label of ‘Physical Description:’.”

This simple case, taking only two parameters (label and method) takes advantage of the SpecBuilder’s defaults. The *setLine()* method can accept up to four parameters, with the third (render type) and fourth (options array) opening up a lot of advanced features and behaviors. You should refer to the VuFind® wiki (see Additional Resources below) for more details; new options are added from time to time, and the documentation will give you the most up-to-date possibilities.

The other two methods(*setTemplateLine()* and *setMultiLine()*) are actually wrappers around *setLine()* which make it more convenient to set up some commonly-used advanced configurations. The *setTemplateLine()* method is by far the most commonly-used option; this retrieves data from a record driver method, but instead of displaying it “raw,” it instead passes it to its own record driver template for additional formatting. This is useful for data fields that need to be linked or labeled in special ways. This method is used to display many of VuFind®’s default fields, which also means that if you want to change the way those fields are displayed, you can simply override the relevant template in your custom theme without having to touch the RecordDataFormatter configuration. Here is an example:

.. code-block:: php

   $spec->setTemplateLine('Series', 'getSeries', 'data-series.phtml');

As you can probably guess, this retrieves data from the record driver’s *getSeries()* method, formats that data using the data-series.phtml template, and then displays the result with a label of “Series:”.

The *setMultiLine()* method is only needed for some rare situations where a single record driver method returns data that needs to be displayed as multiple separate, labeled lines in the output. It allows you to set up a custom PHP function to sort out and format the data. It is rarely needed, and requires more advanced PHP knowledge to understand; if you are interested, you can look at the ‘Authors’ example in the *getDefaultCoreSpecs()* method.

9.3 Example: Adding a Field
---------------------------

This has been the most technical chapter of this guide so far, but even if you do not fully understand all of the underlying technology being discussed, you can still take advantage of the software’s power and flexibility. VuFind® includes several tools for automatically generating code and configurations, so once you understand some common patterns, you can “fill in the blanks” to accomplish important customizations. This section will guide you step by step through the process of indexing and displaying a local custom field.

For the purposes of this example, we will assume that your records use the MARC local notes field 597 subfield a to store donor notes, and that you would like to show them as part of your core metadata with a label of “Donor:”. This is a completely fictitious example; the MARC 59x fields are reserved for local use, and every institution may use them in different ways. If you wish to follow along with this example, you can either create some MARC records with fake data in 597, or you can substitute a different field number in the example to pull data from a field that does exist in your records.

The process of setting up a new field requires only three steps.

9.3.1 Step 1: Index the Data
____________________________

The easiest way to display a new field is to store that data in a field of the Solr index. While VuFind® does have the ability to retrieve details from the raw MARC records stored in its index (using special utility methods included in the SolrMarc record driver), pulling the data out to its own field makes it easier to search and facet using that data, and it provides better uniformity if you also expect to work with the same kind of data from non-MARC sources.


To index the new field, you should:

1.      Create a marc_local.properties file in your $VUFIND_LOCAL_DIR/import directory if you don’t already have one (see section 3.4.3).
2.      Add this line to the marc_local.properties file: *donor_str_mv = 597a*
3.      Reindex all of your records as described in section 3.2.

Note that the “donor_str_mv” field name in the example above takes advantage of the “dynamic field suffixes” configured in VuFind®’s default schema. While adding a new field to Solr usually requires an edit to the schema file (see section 5.1.2), it is possible to define fields based on patterns, so that, for example, any field name ending with “_str_mv” is recognized as a multi-valued string field. VuFind®’s wiki (https://vufind.org/wiki/development:architecture:solr_index_schema#dynamic_field_suffixes) details all of the available dynamic field suffixes.

9.3.2 Step 2: Create a Custom Record Driver
___________________________________________

Now that your index contains data in the donor_str_mv field, you need to tell VuFind® how to read the new field. This requires the addition of a new record driver method. Since we are working with MARC records in this example, the easiest way to set up this method is to extend the SolrDefault record driver in a local code module.

Depending on how you installed VuFind®, you may or may not already have a local module set up; if you do not, or if you are not sure, you can read ahead to section 16.3.3 for more details. For the purposes of this example, we will assume that you have a local module called MyModule. You should substitute “MyModule” with your actual module name in all of the subsequent example commands.

As discussed further in section 17.2.1, VuFind® contains a code generator tool called “extendclass” which can be used to override any core service or plug-in with code in your local module. This saves a lot of time setting up files and configurations. To create the local custom record driver, these commands can be run:

.. code-block:: console

   cd $VUFIND_HOME
   php public/index.php generate extendclass VuFind\\RecordDriver\\SolrMarc MyModule


Note the double backslashes in the class name; because backslash has a special meaning to the Unix command line, it is necessary to “escape” the backslash characters on the command line, or else they will not be passed to the generator correctly.

If successful, you should see output similar to:

.. code-block:: console
  
   Saved file: /…/vufind/module/MyModule/src/MyModule/RecordDriver/SolrMarc.php
   Created backup: /…/vufind/module/MyModule/config/module.config.php.1584707459.bak
   Successfully updated /…/vufind/module/MyModule/config/module.config.php
   Successfully updated /…/vufind/module/MyModule/config/module.config.php
   Successfully updated /…/vufind/module/MyModule/config/module.config.php

Now if you edit $VUFIND_HOME/module/MyModule/src/MyModule/RecordDriver/SolrMarc.php, you will see that the generator has created an empty PHP class for you:

.. code-block:: php

   <?php

   namespace MyModule\RecordDriver;

   class SolrMarc extends \VuFind\RecordDriver\SolrMarc
   {
   }

You simply need to add a method to provide access to the new donor_str_mv field. Edit the file so it looks like this:

.. code-block:: php

   <?php

   namespace MyModule\RecordDriver;

   class SolrMarc extends \VuFind\RecordDriver\SolrMarc
   {
       public function getDonors()
       {
           return $this->fields['donor_str_mv'] ?? [];
       }
   }

All of the Solr fields are exposed to the record driver as part of the *$this->fields* property. The *?? []* syntax simply means “if the requested value is not there, return an empty array instead.”

After making any changes or additions to module.config.php, it is also a good idea to clear your configuration cache, to make sure that your changes take effect immediately:

.. code-block:: console

   sudo rm -rf $VUFIND_LOCAL_DIR/cache/configs/*

9.3.3 Step 3: Create a Custom RecordDataFormatter Configuration
---------------------------------------------------------------

Now that the data is indexed and the record driver can retrieve it, you simply need to tell the RecordDataFormatter view helper to make use of the new record driver method. To do this, you need to extend the RecordDataFormatter’s factory. As of this writing, there is not a code generator to automate this process, but it is simple enough that a generator should not be necessary.

First, make sure you have a custom theme set up, since you will need to register your custom factory in your theme configuration. See section 7.2 for details on creating a new theme. For this example, we assume that your theme is named localtheme.

Next, create a file for your custom factory, called $VUFIND_HOME/module/MyModule/src/MyModule/View/Helper/Root/RecordDataFormatterFactory.p hp. Note that you will have to create a directory to hold this file first, which you can do with:

.. code-block:: console

   mkdir -p $VUFIND_HOME/module/MyModule/src/MyModule/View/Helper/Root

You should fill in the file with this code:

.. code-block:: php

   <?php

   namespace MyModule\View\Helper\Root;

   use VuFind\View\Helper\Root\RecordDataFormatter\SpecBuilder;

   class RecordDataFormatterFactory extends \VuFind\View\Helper\Root\RecordDataFormatterFactory
   {
       public function getDefaultCoreSpecs()
         {
            $spec = new SpecBuilder(parent::getDefaultCoreSpecs());
            $spec->setLine('Donors', 'getDonors');
            return $spec->getArray();
         }
   }

This code takes advantage of PHP inheritance – it calls the core code’s getDefaultCoreSpecs() method to get default configurations, then adds an additional line to display the donors. This way, even if the core code changes in a future VuFind® release to add more fields, we can benefit from those improvements while still adding our additional local field.

By passing additional options, and by manipulating the array inherited from the parent code, it is also possible to change the order of fields, remove unwanted fields, etc. This example is designed to be as simple as possible; see the links under “Additional Resources” for some more advanced discussion and examples.

In any case, now that the factory is built, the last step is to register it in our theme configuration. Edit your $VUFIND_HOME/themes/localtheme/theme.config.php file, and edit it so it looks something like this:

.. code-block:: php

   <?php
   return [
       'extends' => 'bootstrap3',
       'helpers' => [
           'factories' => [
               'VuFind\View\Helper\Root\RecordDataFormatter' => 'MyModule\View\Helper\Root\RecordDataFormatterFactory',
            ],
       ],
   ];

(The helpers section is the important part for the purposes of this example; if you have made other customizations, be sure to reconcile this addition with whatever existing configuration you have).

With all of these changes in place, you should now be able to access a record in the VuFind® web interface and see the “Donors” display (assuming the record has an underlying 597 field). If this does not work, make sure that you replaced all instances of “MyModule” and “localtheme” in code and commands with the appropriate equivalents if your module or theme has a different name. Also, if you had to create a new local code module, make sure that you remembered to restart Apache to load the updated configuration, and double-check that you cleared your configuration cache as described at the end of step 2.

Additional Resources
--------------------

VuFind®’s wiki contains several pages that support and expand upon the information discussed in this chapter: the Record Driver page (https://vufind.org/wiki/development:plugins:record_drivers) and the RecordDataFormatter reference page (https://vufind.org/wiki/development:architecture:record_data_formatter) are of particular interest.

Summary
-------

VuFind®’s record driver system isolates format-specific details in a single place, allowing VuFind®’s other code to be written in a more generic, reusable way. The RecordDataFormatter provides a powerful way to control and customize the way VuFind® displays records. Once you understand these things, you can take control of VuFind®’s presentation of data, and you can add local customizations in a few straightforward steps with the help of VuFind®’s built-in code generation tools.

Review Questions
----------------

1.      What is the responsibility of a record driver in VuFind®?
2.      What is special about the way VuFind® loads record drivers representing Solr records?
3.      What problems inspired the creation of the RecordDataFormatter?
4.      What are the three main methods of the RecordDataFormatter\SpecBuilder, and how do they differ from one another?
