Part 3. Styling and Theming
***************************

Chapter 8. Styling VuFind
#########################


Learning Objectives
-------------------

After reading this chapter, you will understand:

•       How to apply custom styles to your VuFind theme.
•       How to take advantage of CSS frameworks in VuFind.
•       How to use a CSS pre-processor with VuFind.


8.1 Understanding CSS Frameworks
--------------------------------

Since its very earliest releases, VuFind has relied on CSS frameworks to provide a unified look and feel and to simplify some common layout tasks. As of this writing (when VuFind 7.0 was the most recent release), the Bootstrap framework (https://getbootstrap.com/) is used as the foundation for VuFind’s default themes; in earlier releases, Blueprint (http://www.blueprintcss.org/) and YUI (https://yuilibrary.com/) were used. Regardless of which framework is used, its purpose is the same: to establish some core CSS classes that enable useful functionality like responsive design, grid layout, etc.

Going in depth on any particular framework is beyond the scope of this book; however, understanding the framework you are working with will help you to read and customize the templates in your theme. It should be easy to find a primer for any major framework using the search engine of your choice; just be sure to read about the same version of the framework that VuFind is using, since some frameworks can introduce significant changes when they introduce major new versions. (For example, Bootstrap 3 and Bootstrap 4 are quite different from one another; VuFind still uses Bootstrap 3 as of this writing, but it is possible that Bootstrap 4 may be adopted in the future).

If you are not sure what framework your VuFind theme is using, the theme name (or the name of a parent theme) may help to clarify this. (For example, VuFind’s bootstrap3 theme is the basic implementation using the Bootstrap 3 framework; there are some more heavily styled sub-themes of bootstrap3, such as bootprint3 and sandal, but if you examine the theme.config.php file in either of these themes, you will see that it extends bootstrap3).

8.2 Understanding CSS Preprocessors
-----------------------------------

While CSS is a very powerful language for applying styles to web pages, maintaining CSS files can sometimes be labor intensive and error-prone. Due to limitations in the language, some tasks require the creation of extremely verbose selectors or extensive copying-and-pasting of values (such as color definitions). CSS Preprocessors were created to address some of these limitations.

A CSS Preprocessor defines a new language (usually a superset of CSS itself) which adds new features like nested blocks, variables, etc. The preprocessor also includes a tool (called a compiler) which interprets files in the new language and translates them into regular CSS so that they can be used on the web. Using a preprocessor introduces a small trade-off: you have a more powerful language for styling your website, but you have to remember to run the compiler to build your CSS when you want to deploy changes.

As of this writing, VuFind’s default themes make use of the LESS preprocessor, and VuFind includes tools for compiling LESS into CSS using either pure PHP (which is slower, but has fewer dependencies) or Javascript (which is faster but requires installation of the Node.js environment on your server). All of VuFind’s LESS files are also distributed in SASS/SCSS format (a different, but very similar, preprocessor) to accommodate users who already use that format in their development workflows. Because front-end development is a fast-evolving area, it is possible that additional preprocessors may be supported in the future, and that some of the currently-supported formats may eventually be abandoned.

As with CSS frameworks, going in depth on CSS preprocessors is beyond the scope of this text, but tutorials should be available online for LESS and other popular formats.

8.3 Custom CSS vs. Custom LESS
-------------------------------

If you want to add some custom styles to your local theme, you should consider whether you wish to add a custom CSS file or a custom LESS file. Adding a custom CSS file has the advantage of simplicity; you simply create and register a file, and changes to that file will automatically take effect. Adding a custom LESS file lets you take advantage of the power of CSS preprocessing, but it has a more complicated setup process and will require you to remember to compile CSS when making changes. This section describes both approaches.

8.3.1 Adding a Custom CSS File
_______________________________

To add a custom CSS file to your theme, you can simply edit your theme’s theme.config.php file, and add a new CSS filename to the ‘css’ element of the configuration. All CSS files listed in this array will be loaded on every page of VuFind’s interface, so this is the best way to make global changes. Because of VuFind’s theme inheritance, it is important to give your CSS file a unique name; if you give it the same name as one of the CSS files in a parent theme, it will completely replace that existing file rather than being added to it.

As an example, assume that you have created a custom theme named “localtheme” as described in section 7.2 above, and you want to change the body background color to a pale gray color.

First, you should edit your $VUFIND_HOME/themes/localtheme/theme.config.php file; by default, it should look like this:

.. code-block:: console

   <?php
   return [
       'extends' => 'bootstrap3'
   ];


You should add a comma on the end of the ‘extends’ line, and put a ‘css’ line below that pointing to an array of filenames (which for now will include just one file):

.. code-block:: console

    <?php
    return [
        'extends' => 'bootstrap3',
            'css' => ['myinstitution.css']
      ];

This tells VuFind to add a CSS file called myinstitution.css to every page of its interface; we chose the name myinstitution.css to avoid any possible naming conflict with the core themes (of course, you could replace “myinstitution” with the actual name of your institution if you wished). You only need to specify the filename itself, not any path information; VuFind will search for this filename in your theme’s css folder, and should it fail to find it, it will also search through all of the parent themes.

In order to ensure that VuFind actually finds something when it does its search, you should also create the expected file by editing $VUFIND_HOME/themes/localtheme/css/myinstitution.css. You can paste in this content:

.. code-block:: console

   body {
    background-color: #d0d0d8;
    }

Now if you refresh VuFind in your browser, you should see that the local theme’s default background color has changed.

8.3.2 Adding a Custom LESS File
_______________________________

VuFind’s provided themes are set up so that all of the LESS files provided are compiled into a single CSS file called “compiled.css.” This setup makes adding a new LESS file a little bit complicated. Fortunately, the sample theme created by the generate command (see section 7.2) creates some example LESS files for you, providing a helpful foundation for you to build upon.

If you look in $VUFIND_HOME/themes/localtheme/less after generating the theme, you will see three files: compiled.less, which is the top-level file that VuFind will use to compile the LESS into CSS, based on configuration inherited from a parent theme. All this file does is include custom.less, which is the place where you can put your own custom styles.

If you edit custom.less, you will see that its first line is:

.. code-block:: console

   @import “bootstrap”;

This pulls in the default Bootstrap framework styles, which you will need to take advantage of the framework and to make sure that default VuFind templates display correctly. You should leave this line alone.

Everything else in custom.less is an example, and you are free to change or remove it. The provided example shows how to define some variables (like “@active-orange” and “@dark-green”) for internal use, and also how to override some core Bootstrap and VuFind variables (like @brand-primary and @body-bg) to change the way the theme looks without having to build CSS stanzas. There are also some more specific example styles below the variables, and the file ends by demonstrating that you can use @import statements to pull in additional files if you want; the home-page.less file is an example of this capability.

If you wanted to implement the same background color change that was used as an example in 8.3.1, you could accomplish it here by editing a single variable and then recompiling the LESS.

First, edit $VUFIND_HOME/themes/localtheme/less/custom.less, and change this line:

.. code-block:: console

   @body-bg: #5ab48a;

to

.. code-block:: console
  
   @body-bg: #d0d0d8;
   

This will have exactly the same effect as the CSS file override described earlier; however, rather than adding a new CSS rule to override an earlier rule, changing this variable enables you to actually change the CSS output created when the LESS compiler processes the files found in the parent themes.

Before you will see the results of your change, you must compile the LESS into CSS. You have two options for this: the slow PHP compiler, or the faster Javascript compiler.

8.3.2.1 Compiling LESS Using PHP
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To use the PHP-based compiler, simply run these commands:

.. code-block:: console

   cd $VUFIND_HOME
   php util/cssBuilder.php

8.3.2.1 Compiling LESS Using Javascript
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To use the Javascript-based compiler, you will need to install Node.js and grunt on your system. This is usually a matter of installing the nodejs package with your platform’s package manager, and then running:

.. code-block:: console

   cd $VUFIND_HOME
   npm install -g grunt-cli
   grunt less

Once grunt is installed, you can compile your LESS with:

.. code-block:: console

   cd $VUFIND_HOME
   grunt less

Additional Resources
--------------------

The Bootstrap 3 documentation is available at https://getbootstrap.com/docs/3.3/. You can learn more about LESS at the language’s official website (http://lesscss.org/). VuFind’s use of CSS preprocessing is discussed in more detail on this wiki page: https://vufind.org/wiki/development:architecture:less.

Summary
-------

VuFind’s themes are built using popular CSS frameworks, establishing useful conventions and basic functionality. VuFind also uses CSS preprocessing to work around some of the limitations of CSS when designing its styles. When building your own theme, you can choose to add simple CSS files, or you can do a bit more work to access the full power of preprocessing.

Review Questions
----------------

1.      Why does VuFind use CSS frameworks?
2.      Why does VuFind use a CSS preprocessor?
3.      What are the advantages and disadvantages of using custom CSS vs. using custom LESS?
