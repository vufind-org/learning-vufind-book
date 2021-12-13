###################################
Chapter 7. Creating a VuFind® Theme
###################################

Learning Objectives
-------------------

After reading this chapter, you will understand:

•       How to create your own VuFind® theme.
•       How to override view templates.
•       How to read and understand the conventions of VuFind®’s view templates.
•       How to use some of VuFind®’s advanced theme configuration options


7.1 Introduction to Themes
--------------------------

VuFind® is designed using a model-view-controller architecture (as discussed further in section 16.2). One of the key features of this design is that the code for how VuFind® does things (the “business logic”) is separated from the code for how VuFind® displays things (the “view templates”). This is a powerful separation, since it means that VuFind®’s look and feel can be changed independently of its core functionality. It also means that it is possible to provide multiple presentations of the system.

VuFind® includes a theme system which makes it easy to bundle all of VuFind®’s user-facing pieces (view templates, Javascript code, CSS styles, images) in a single package. This theme system also supports the idea of inheritance, so you can create a “sub-theme” which can override some parts of a parent theme while inheriting other parts as-is. For example, if you like one of VuFind®’s default themes but just want to override some CSS styles and the text of the footer, you can easily create a sub-theme that contains a custom CSS file and a custom footer template, and all of the other functionality will carry through from the parent theme.

VuFind® includes a root theme, from which all other themes extend; this theme contains some universally-needed templates (such as pre-formatted email messages, rules for exporting records into bibliographic management systems, help screens, etc.) and some shared fonts and images. The other available themes will depend on the version of VuFind® you are using; as web technology evolves quickly, VuFind® has kept up with progress by providing several different theme options over the years, leveraging different frameworks. While the names may change over time, the basic principle is that there is always a base theme that contains all of the templates used for rendering VuFind®’s many pages, and then there are some sub-themes which apply extra CSS to this foundation in order to provide different stylistic options. You can extend from any of these options to build your own theme; it is simply a matter of which option provides the most work that you can reuse in order to keep your local customizations as small and simple as possible.

This chapter will show you how to create your own themes, how to understand some of the code you will see in the existing themes, and how to configure VuFind® to use different themes.

7.2 Using the Theme Generator
-----------------------------

While creating a theme is fairly straightforward (just a matter of creating a directory containing a configuration file and some subdirectories under $VUFIND_HOME/themes), VuFind® provides a command-line tool to automatically build a sample theme for you, so you have an easy starting point. Just decide on a theme name (for the example here, we will use localtheme), and run:

.. code-block:: console

   php $VUFIND_HOME/public/index.php generate theme localtheme

This will create a new $VUFIND_HOME/theme/localtheme folder containing some example files for you to modify. It will also modify your $VUFIND_LOCAL_DIR/config/vufind/config.ini to make your new theme the default option, and to turn on a theme selection control to allow users to switch back to the previous setting. (See 7.6 below for information on how to change this if you so desire).

7.3 Understanding theme.config.php
----------------------------------

Every theme in VuFind®, including the one you created with the generator in 7.2, will have a theme.config.php file in its root directory. At a bare minimum, this file simply defines a parent theme through the ‘extends’ setting, establishing the aforementioned inheritance mechanism. The file can also be used to define globally-loaded resources like extra Javascript and CSS files, to set a ‘favicon’ (the small graphic that browsers associate with your site in the address bar and bookmarks), or to set up custom view helpers (discussed further in 7.5).

It is worthwhile to take a look at some of the theme.config.php files included with VuFind® to gain a better understanding of the available options and to see some examples of how things are formatted. The ‘root’ theme simply defines commonly-used view helpers. Other themes extend this with the necessary Javascript frameworks and CSS styles.

Once you have established your theme, you probably won’t come back and edit theme.config.php very frequently unless you are adding new styles or building custom view helpers. These situations will be discussed in more detail later in the book.

7.4 Understanding Templates
---------------------------

7.4.1 Basic Technology
______________________

VuFind®’s views are built in native PHP, but they are loaded using the laminas-view component, which sets up some conventions and adds some useful features. PHP already gives you the ability to mix HTML markup with dynamic computer code. The laminas-view component provides a standard way for the business logic code of VuFind® to pass variables to the view templates. The job of a template designer is simply to set up a useful presentation of those variables.

7.4.2 Template Locations
________________________

VuFind®’s templates can be found in the templates directory of its themes. When VuFind® needs to display a template, it first looks in the currently active theme’s templates directory. If no match is found there, it tries the parent theme’s templates directory. It continues searching from parent to parent until it either finds the template it needs, or it runs out of themes to search. If you need to find a particular template, you can always do the same thing: look at theme.config.php to identify the parent of your theme, and keep tracing your way back until you find where the templates reside. As mentioned earlier, VuFind® has some themes that are CSS-only, and others that contain the bulk of the templates.


7.4.3 Template Naming
_____________________

Most of the folders under the templates directory are named after the controllers or plug-ins that they are associated with. Some templates are “partials” – rules for rendering commonly-used chunks of content that may be used in multiple places. For example, the header and footer displayed on every page of the interface are split out into their own header.phtml and footer.phtml files, to make these commonly-customized regions easier to override independently.

It is not always easy to guess exactly which file will contain the part of the interface that you need to override; it is very helpful to use an editing tool with a “search within files” feature when working on VuFind®, since it is usually fairly straightforward to find the template you wish to edit by searching for distinctive CSS classes or text labels found within the page you wish to modify.

7.4.4 Reading a PHP Template
____________________________

The more you understand about both HTML and PHP, the more comfortable you will be working with VuFind®’s templates, and the more powerfully you can modify them. However, even if your understanding of these things is fairly limited, you should still be able to recognize patterns and make simple changes like rearranging content, adding labels, etc.

Most of a template is just plain HTML. However, you may see some :code:`<?php … ?>` blocks containing PHP logic, and some :code:`<?= … ?>` blocks used for displaying variables inline. For example, you might see something like this:

.. code-block:: php

   <?php if (isset($title)): ?>
     <h1><?=$title?></h1>
   <?php endif; ?>

The :code:`if .. endif` block checks to see if a variable called :code:`$title` has been set. If the variable is present, the inside part of the block is triggered, creating an :code:`<h1>` tag and displaying the value of :code:`$title` within it.

This is a greatly simplified example, but it demonstrates the basic flavor of templates.

7.5 Understanding View Helpers
------------------------------

While it is possible to write large amounts of PHP logic directly into template files, this is generally discouraged, as it makes templates harder to read; the focus of a template should be on presentation rather than logic. However, sometimes there is a need to do some complex data processing on a variable before displaying it, or there may be a repetitive task (like rendering a formatted table) that is better done with reusable code than with copy-and-paste. In these situations, a view helper may be useful.

View helpers are a feature of the laminas-view component – they provide a mechanism for hooking up PHP classes full of logic with your view templates in a concise way. The laminas-view component includes a number of useful helpers for common tasks, and VuFind® adds many additional specialized options.

7.5.1 Internationalization
__________________________

Two of the most common  view helpers you will see in VuFind®’s templates are :code:`$this->translate()` and :code:`$this->transEsc()`. These are part of VuFInd’s internationalization system – they make it possible for users to experience the VuFind® user interface in a variety of languages. Any text that is passed to these functions is looked up in the language files found under $VUFIND_HOME/languages, and the equivalent text from the user’s selected language is displayed in place of the input. Understanding this is important for several reasons. First of all, if you are searching through templates trying to find a particular piece of text and cannot find it, it may be because the text actually comes from one of the language files, and is represented in the templates as an abbreviated token. Secondly, if you operate a VuFind® instance that supports multiple languages, you will need to use these view helpers and populate custom language files (under $VUFIND_LOCAL_DIR/languages) to ensure a correct experience for users of all languages. Finally, when reading templates, it’s important to understand what the translate helpers are doing.

7.5.2 Escaping
______________

It is also important to understand the difference between :code:`$this->translate()` and :code:`$this->transEsc():` the plain “translate” helper just looks up a string and outputs it as-is; the “transEsc” helper is the same as “translate” but adds an additional step of HTML escaping, making sure that the text is safe to output as part of an HTML document. (For example, this makes sure that text containing < and > characters does not get misinterpreted as an HTML tag). There is also a :code:`$this->escapeHtml()` helper for escaping text without translating it first, and a :code:`$this->escapeHtmlAttr()` helper that applies extra-strict escaping to values that will be presented as HTML attributes. For security and reliability, it is important to be disciplined about consistently escaping values in templates.


7.6 Understanding Layouts
--------------------------

One of VuFind®’s most important templates can be found in layout/layout.phtml under the templates directory. This layout template is used to provide the overall structure of every page in VuFind®’s interface. The other templates are used to fill in the “content” block at the center of this template. If you need to change the overall structure of pages displayed on your VuFind® instance, this is the template you will want to customize. It is also important to understand that the layout template is always rendered last. VuFind® first processes the inner content template, then inserts it into the layout. The :code:`$this->layout()` view helper can be used to share information between the layout and the inner templates, but of course this sharing can only happen in one direction: values set inside the layout helper by inner templates can be accessed by the layout, but the reverse is not possible. By the time the layout sets a value, the inner template has already been fully rendered and is no longer in a position to receive that information.

7.7 Configuring Multiple Themes
-------------------------------

There are several settings in the [Site] section of config.ini that you can adjust to control how VuFind® loads themes.

Most obvious is the “theme” setting: this is the theme that will be loaded by default when a user accesses VuFind® for the first time.

There is also a “mobile_theme” setting which can be used to load a different theme when a mobile device is detected. This used to be more important when it was popular to display completely different interfaces on mobile vs. desktop devices; now that “responsive design” is the norm, there is very little reason to use this option, and VuFind® no longer includes a mobile-specific example theme.

The “alternate_themes” setting allows you to create a list of themes that can be accessed by passing a ?ui= parameter on the end of your VuFind® URL. This is a comma-separated list of themes, which are represented as colon-separated pairs. In each of these pairs, the first value is the text that can be passed in as the ui parameter, and the second value is the name of the actual theme to load when that value is passed in. For example:

:code:`alternate_themes=my1:MyTheme1,my2:MyTheme2`

would load MyTheme1 if a user added ?ui=my1 to the default VuFind® URL, and would load MyTheme2 if they instead passed ?ui=my2.

Finally, the “selectable_themes” setting creates a drop-down list that allows users to switch back and forth between themes. It is also a comma-separated list of colon-separated pairs. These pairs consist of the shorthand name for a theme (i.e. the first part of one of the alternative_themes pairs, or else “standard” for the default theme, or “mobile” for the mobile-specific theme). The order of the list controls the order of the drop-down displayed in VuFind®.

Most VuFind® administrators will only need to provide a single theme, but if you want to offer choice to your users (for example, “light” and “dark” themes for users with different vision preferences), setting up alternate_themes and selectable_themes can be helpful. It may also be useful to set up alternate_themes without selectable_themes if you wish to provide access to a different look and feel without advertising it to most users (for example, if you want to provide beta or testing functionality that is only available to a limited audience).

Additional Resources
--------------------

A video covering many of the topics from this chapter is available through the VuFind® website (https://vufind.org/wiki/videos:creating_themes). Further information can be found on the User Interface Customization wiki page (https://vufind.org/wiki/development:architecture:user_interface).

Summary
-------

VuFind® themes allow you to put all of your presentation-related resources in a single place. VuFind®’s “theme inheritance” makes it possible to make a few small changes while inheriting most of the features of a pre-built core theme. Tools are available to automate the creation of new themes. With an understanding of a few configuration settings, HTML and PHP, it is possible to take control of VuFind®’s look and feel to meet your local needs.

Review Questions

1.      What are three things you can change or add through the theme.config.php file?
2.      In a template file, what does a :code:`<?php … ?>` tag mean?
3.      What can go wrong if you forget to escape text from a variable in a template?
4.      What is the difference between the “alternate_themes” and “selectable_themes” configuration settings?

