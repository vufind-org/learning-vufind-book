###########################
Chapter 17. VuFind Plug-Ins
###########################

After reading this chapter, you will understand:

•       How and why VuFind uses plug-in code.
•       Which types of plug-ins are available in VuFind.
•       How to generate plug-ins on the command line.


17.1 VuFind’s Plug-In Architecture
----------------------------------

One of VuFind’s design goals is to be as extensible as possible, and one of the ways in which it implements extensibility is to use plug-in code. All of VuFind’s pluggable code follows the same basic strategy: define a common interface that broadly defines a piece of required functionality, offer multiple implementations of the interface to meet different needs, and use configuration to load the appropriate implementation based on user preferences. Plug-ins fall into two broad categories: those that allow a function to be fulfilled by different technologies by exposing a generic interface to higher-level VuFind code (for example, authentication handlers, session handlers, ILS drivers, etc.); and those that provide “hooks” for custom functionality (for example, recommendation modules, related record plug-ins, DOI handlers, etc). Regardless of the behavior or purpose of the plug-in, the general mechanisms for configuring and accessing it are the same.

As discussed in section 16.4, Laminas offers a service manager component which offers a useful abstraction for loading and building complex objects “on the fly” based on configuration. The service manager provides the foundation for all of VuFind’s plug-ins.

VuFind actually uses a large number of service managers, arranged in a two-level hierarchy. There is the top-level service manager which manages unique and globally-used services as well as all of the lower- level service managers, each of which in turn manages a family of related plug-ins. To fetch a specific plug-in, code first needs to request the appropriate plug-in manager from the top-level service manager, and then the plug-in itself can be requested from the plug-in manager. To differentiate between the two different levels of managers, the lower-level managers are usually referred to as “plug-in managers” while the top-level manager is simply called “the service manager.”

The most meaningful difference between the top-level service manager and a lower-level plug-in manager is simply how each is configured; they all provide the same dependency injection and container functionality, but they are set up to build and manage different types of objects. Using a dedicated plug- in manager for each type of plug-in offers several benefits over simply loading everything into the top- level service manager:

•       We can better organize the application’s service configurations, making it more readable and understandable.
•       We can improve efficiency by only loading configurations for specific plug-in types when they are actually needed.
•       We can take advantage of the service manager’s capability to assign short, convenient aliases to longer service names without having to worry about collisions if two different types of plug-in need to use the same short alias name – each plug-in manager maintains its own independent set of aliases, to be used only within the context of that plug-in manager.
•       We can reduce the odds of accidentally loading the wrong type of object in the wrong context by storing different families of plug-ins in separate places.
•       For services that need to dynamically load a particular type of plug-in, we can inject a specific plug-in manager as a dependency rather than injecting the top-level service manager, which is a bad design practice (because it offers more access to resources than any single class is likely to need, making relationships between parts of the application less clear).


All of the plug-in managers are registered as services in the top-level service manager, using their class names as their service names. All of the plug-ins are registered in the corresponding plug-in managers using both their class names and shorter, more human-readable aliases (which make them easier to load based on configuration). Each plug-in manager is implemented as a subclass of the Laminas service manager, with default configurations built into the code. These default configurations can be overridden and extended through local module configuration, making it possible to easily add or replace any kind of plug-in. The command line generator tools (discussed in the next section) can be used to automate most of the configuration work to take advantage of this setup. A few plug-in types are actually part of the Laminas framework (controllers, controller plug-ins and view helpers); the rest are specific to VuFind.

Several components of VuFind discussed elsewhere in this text utilize this plug-in architecture: view helpers (section 7.5), record drivers (section 9.1), recommendation modules (chapter 14), and most of the components that make up search backends (chapter 15). There are many more plug-in types within the VuFind code; see “Additional Resources” below to learn more about them.

There is no question that VuFind’s plug-in manager and service manager code and configuration is complex, and it can take some time and experience to fully understand it. However, this complexity is designed to make a lot of higher-level tasks easier and simpler. Only experienced VuFind developers need to master this; newcomers can simply take advantage of it to build custom code more easily. Do not be concerned if you do not understand everything yet; you can take advantage of the power of plug- ins without fully understanding how they are implemented in VuFind’s core code. The next section will show you how to easily build your own plug-ins without having to touch any low-level VuFind code or configuration.

17.2 Command Line Plug-In Generator Tools
-----------------------------------------

While the Laminas service manager and plug-in managers are powerful tools, setting up or extending a plug-in can require a lot of work: creating the plug-in class itself, creating a factory to build the plug-in, and configuring everything in the module system. Fortunately, VuFind includes command-line tools that automate most of this work, ensuring that everything ends up in the right place, and allowing you to focus on the more interesting work of writing meaningful code.

There are a couple of things you’ll need to remember to make the most of the generators:

• Make sure you have all of your environment variables, including VUFIND_LOCAL_MODULES, exported correctly when running the generators, or else they will not be able to find the right place to put your custom code. See section 16.3.3 for more discussion of local modules.

• Many of the commands require you to type fully qualified PHP class names. Because these class names include backslash (\) characters, and these characters have special meaning on the command line, you will need to escape them by doubling them. So be sure you type (for example) “VuFind\\Recommend\\SideFacets” and not “VuFind\Recommend\SideFacets” when you list a class name as a parameter.



17.2.1 Replacing/Extending a Plugin (the “extendclass” generator)
_________________________________________________________________

If you want to extend or override an existing plug-in, VuFind’s “extendclass” generator will help. You simply provide it with the name of the core VuFind class that you wish to extend, and the name of a module where you would like to put the new subclass, and it will set up the rest. By default, it will reuse whatever factory is already defined in the core to build your new subclass, but if you also wish to create a new factory, you can add the “--extendfactory" option to create a subclass of the factory as well as the plug-in itself.

The examples below assume a local custom module with the name “MyModule.”

Example 1: Extend the SideFacets Recommendation Module, Keep the Existing Factory

.. code-block:: console

    php $VUFIND_HOME/public/index.php generate extendclass VuFind\\Recommend\\SideFacets MyModule

Example 2: Extend the Koha ILS Driver, Use a Custom Factory

.. code-block:: console

    php $VUFIND_HOME/public/index.php generate extendclass --extendfactory VuFind\\ILS\\Driver\\Koha MyModule

Example 3: Extend an ILS Driver
See section 9.3.2 for another example of this code generator in action.

17.2.2 Creating a Plugin (the “plugin” generator)
_________________________________________________

If you want to create a new plug-in, VuFind’s “plugin” generator will do the job. You simply tell it the class name that you wish to create, and it will infer from the namespace of the class which module you want to update and which plug-in manager needs to be updated to register it. If you provide a class name by itself, the generator will also build an accompanying factory class to build the plug-in. If you provide the name of an existing factory as the command’s second parameter, that factory will be used to construct the object in the generated configuration, and no additional factory class will be built.

The examples below assume a local custom module with the name “MyModule.”

Example 1: Create a New Record Driver and Accompanying Factory

.. code-block:: console

    php $VUFIND_HOME/public/index.php generate plugin MyModule\\RecordDriver\\MyRecordType

Example 2: Create a New Recommendation Module; Use an Existing Factory
The next section (17.3) consists of an extended example matching this scenario.

17.3 Example: Building a New Recommendation Module
__________________________________________________

As discussed in chapter 14, recommendation modules provide a way to supplement search results with additional useful information. It is often useful to build custom recommendation modules to provide local information or to augment custom search handlers. This section will demonstrate how to build a bare-minimum recommendation module which simply causes some custom text to be displayed; it can be used as the basis for more complex plug-ins.

17.3.1 Building the Recommendation Module Class
_______________________________________________

This example will assume that your local module is set up and named MyModule, and that the recommendation module class you want to create will be named MyModule\Recommend\LocalText. Every part of this name is meaningful to VuFind’s generator tool: the first part of the namespace (MyModule) tells it that the class needs to be created inside the MyModule module; the middle part of the namespace (Recommend) tells it to create a recommendation module, since every VuFind recommendation module is in the “VuFind\Recommend” namespace; the final part specifies the actual class name being created.

Because our example is going to be very simple and will have no external dependencies, we do not need to build a custom factory for it. Instead, we want to use the standard, framework-provided Laminas\ServiceManager\Factory\InvokableFactory which (as discussed in section 16.4) simply constructs objects without passing any parameters to them.

Thus, the command to actually set up our recommendation module is:


.. code-block:: console

   php $VUFIND_HOME/public/index.php generate plugin MyModule\\Recommend\\LocalText Laminas\\ServiceManager\\Factory\\InvokableFactory


The output will resemble this:

.. code-block:: console

   Saved file: /…/vufind/module/MyModule/src/MyModule/Recommend/LocalText.php
   Created backup: /…/vufind /module/MyModule/config/module.config.php.1588872149.bak
   Successfully updated /…/vufind /module/MyModule/config/module.config.php
   Successfully updated /…/vufind /module/MyModule/config/module.config.php


You will now have an empty recommendation module set up in $VUFIND_HOME/module/MyModule/src/MyModule/Recommend/LocalText.php:

.. code-block:: php

   <?php

   namespace MyModule\Recommend;

   class LocalText implements \VuFind\Recommend\RecommendInterface
   {
   }


Now all that is left is to fill in the code to fulfill the requirements of the RecommendInterface, which every recommendation module needs to implement. This interface contains three methods: setConfig(), which processes configuration settings passed in from the .ini file; init(), which can make adjustments to search backend parameters prior to performing the search; and process(), which can extract data from search results after the search has been completed. For our current example, none of these methods need to do any work, but we still need to define them to comply with the interface. We can simply add them to the file, so that it looks like this:

.. code-block:: php

   <?php

   namespace MyModule\Recommend;

   class LocalText implements \VuFind\Recommend\RecommendInterface
   {
       public function setConfig($settings)
       {
       }

       public function init($params, $request)
       {
       }

       public function process($results)
       {
       }
   }

Additional functionality can be added to these stub functions in the future if the need should arise, but simply defining them so that they do nothing is good enough for the purposes of this example

17.3.2 Creating the Recommendation Module Template
__________________________________________________

Every recommendation module needs a template file to serve as its view component so that it can be displayed on screen. By convention, these match the name of the class and are stored in the Recommend folder of the template directory. If you set up a local theme named “localtheme” as described in section 7.2, you could edit the file $VUFIND_HOME/themes/localtheme/templates/Recommend/LocalText.phtml to set up the view for your recommendation module. For example, try something like this:

.. code-block:: php

   <p>If you need more help, be sure to talk to a librarian!</p>

17.3.3 Activating the Custom Module
___________________________________

Now, the only thing left to do is to make your new recommendation module visible. For example, you could edit your $VUFIND_LOCAL_DIR/config/vufind/searches.ini (remember to copy it from $VUFIND_HOME/config/vufind/searches.ini if you don’t already have one) and add this to the [General] section:


.. code-block:: properties

   default_top_recommend[] = LocalText


Now, when you perform a search, you should see your custom text above the search results.

17.3.4 Taking It Further
________________________

While this example has now served the purpose of showing how you can create a very simple plug-in, we can make a few more adjustments to this example to also show off more of the power of custom recommendation modules. So far, we have shown that you can build a custom class and template to display some text, but the custom class doesn’t actually do anything. Let’s revise it to make it configurable. Change the PHP file so that it contains this content:

.. code-block:: php

   <?php
    namespace MyModule\Recommend;

    class LocalText implements \VuFind\Recommend\RecommendInterface
    {
        protected $name = 'a librarian';

        public function setConfig($settings)
        {
            if (!empty($settings)) {
               $this->name = $settings;
            }
        }

        public function init($params, $request)
        {
        }

        public function process($results)
        {
        }

        public function getName()
        {
            return $this->name;
        }
    }

And change the template so that it contains this content:

.. code-block:: php

    <p>If you need more help, be sure to talk to <?=$this->escapeHtml($this->recommend->getName())?>!</p>

Now, if you refresh your search results, you will still see the same text as before… but you have gained the ability to override the name being displayed through the configuration file. Try editing searches.ini like this:

.. code-block:: properties

    default_top_recommend[] = "LocalText:Your Name Here"

Now if you refresh the search results, “a librarian” will be replaced with “Your Name Here.” So how did this work?

In the LocalText PHP class, we added a property called $name. This is initialized to “a librarian,” so it has a default value.

The setConfig() method is called when the recommendation module is initialized; it is passed any configuration settings that are attached to the recommendation module name with a colon. In other words, when we configure “LocalText: Your Name Here” in searches.ini, VuFind passes the text “Your Name Here” to the setConfig method. The logic here checks if the incoming text is non-empty, and overrides the default value of the $name property when appropriate.

We also defined a public getName() method, which simply provides the current value of the $name property.

In the view template, the recommendation module object is exposed as a value called $this->recommend. Thus, we can define any public methods we like in the PHP code and then access those methods from the template. Thus, *<?=$this->escapeHtml($this->recommend->getName())?>* in the template provides the connection where the value from our configuration file can be surfaced in the user interface. The *$this->escapeHtml()* call wrapped around *$this->recommend->getName()* simply ensures that the provided text is properly formatted as HTML, in case it contains special characters like <, > or &.

Hopefully you can see how this offers you a great deal of power to use PHP code to retrieve information from various sources and then expose it to the user through the template. If you examine the existing recommendation modules that ship with VuFind, you will see more advanced examples of how to leverage third-party APIs, make decisions based on the contents of search results, and even manipulate the parameters used to perform the search.

Additional Resources
--------------------

The VuFind wiki has a page summarizing all available plug-in types and providing advice on how to build them: https://vufind.org/wiki/development:plugins.
The tutorial video at https://vufind.org/wiki/videos:code_generators_1 includes an example of building a custom recommendation module.

Summary
-------

The Laminas service manager component provides conventions that allow VuFind to support extensible and pluggable functionality throughout the application. Nearly any piece of VuFind can be replaced by overriding configurations, and additional functionality can be plugged in in a similar way. Code generator tools included with the application automate the process of creating files and editing configurations, allowing developers to focus on writing code rather than setting up boilerplate.

Review Questions
----------------

1.      What are some reasons that VuFind uses plug-in managers instead of simply registering all plug-in code directly within the top-level service manager?
2.      In what situation would you want to generate a new factory for a plug-in? When would it be better to use an existing factory?
3.      What is wrong with the following command?

.. code-block:: console

  php $VUFIND_HOME/public/index.php generate plugin MyModule\RecordDriver\MyRecordType
