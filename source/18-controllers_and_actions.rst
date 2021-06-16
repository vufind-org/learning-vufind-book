###################################
Chapter 18. Controllers and Actions
###################################

Learning Objectives
-------------------

After reading this chapter, you will understand:

•       The purpose of routes and controllers.
•       VuFind’s controller conventions.
•       How to add functionality to VuFind’s web interface.


18.1 VuFind’s Controllers and Actions
As discussed in section 16.2, VuFind uses the laminas-mvc package to implement a model-view-controller architecture. Controllers are the part of the code that make things happen in VuFind.

When a user accesses one of VuFind’s URLs, the laminas-router system is triggered. The router component compares the text of the request URL against a set of rules configured in VuFind’s Laminas modules in order to figure out which piece of controller code to dispatch.

For example, if the user enters http://localhost/vufind/Example/Page, they will most likely match VuFind’s default routing rule, defined in $VUFIND_HOME/module/VuFind/config/module.config.php:

.. code-block:: console 

   'default' => [
    'type'    => 'Laminas\Router\Http\Segment',
    'options' => [
        'route'    => '/[:controller[/[:action]]]',
        'constraints' => [
            'controller' => '[a-zA-Z][a-zA-Z0-9_-]*',
            'action'     => '[a-zA-Z][a-zA-Z0-9_-]*',
        ],
        'defaults' => [
            'controller' => 'index',
            'action'     => 'Home',
        ],
    ],
   ],

This particular route uses a ‘type’ of ‘Laminas\Router\Http\Segment’, which means that the route definition in ‘route’ can contain “segments” that start with colons (like :controller and :action), allowing parts of the user’s URL to be extracted and passed along to other parts of the code. Some of the route definition is enclosed in brackets, making it optional. Defaults are provided to give values to the segments in case the user omits them entirely – so with this route definition, accessing simply http://localhost/vufind is effectively the same as accessing http://localhost/vufind/index/Home. 

To learn about the Laminas router component in more detail, see the documentation at https://docs.laminas.dev/laminas-router/routing/.

To continue the example of accessing the URL of http://localhost/vufind/Example/Page, the default route defined above would extract a ‘controller’ value of ‘Example’ and an ‘action’ value of ‘Page’. These route values have special meanings when used in the laminas-mvc framework that VuFind is built on: they control which PHP code gets executed.

First, the controller value is used to look up a controller class in the controller plugin manager (see chapter 17 for more on plugin managers). By convention, VuFind’s controllers are set up with short names as aliases to make routing configuration more readable – thus, if VuFind had a VuFind\Controller\ExampleController, it would also have an alias configuration allowing ‘Example’ to be used as a shorthand for retrieving the controller from the plugin manager.

If no matching controller can be found, VuFind will throw a 404 not found error… but if a controller exists, the action parameter from the route will be used to call a matching method on the controller. Controller actions must be named as an action name, followed by the word Action – so in the current example, VuFind would attempt to call the pageAction() method on the ExampleController.

Controller action methods most commonly return a Laminas\View\Model\ViewModel object, which is a container that is used to pass data along to a view template for rendering. Most controller actions have corresponding view templates in the theme (for example, the pageAction of the ExampleController would expect to render a template named example/page.phtml). Any data elements that are inserted into the ViewModel object will be available to the view template as local variables that can be used for display.

In some special cases, controller actions may instead return a Laminas\Http\Response object representing a fully-formed HTTP response. This bypasses the usual view rendering behavior and can be useful in situations where special actions are needed (like a forced redirect to a different page) or where different types of content need to be returned (like when rendering an image, or providing a download of a binary file like a PDF).

Controllers are discussed in more detail in the Laminas documentation at https://docs.laminas.dev/laminas-mvc/controllers/.

Most of VuFind’s controllers extend the VuFind\Controller\AbstractBase class, which provides some helpful convenience methods for common VuFind-related activities. For example, there is a createViewModel() method which constructs a ViewModel object with some useful pre-populated values. In general, when adding new functionality to VuFind, it is useful to examine existing controllers and actions providing similar functionality, as there may be some patterns observed there that can be usefully emulated.

18.2 Example: Adding a Page to VuFind
--------------------------------------

The http://localhost/vufind/Example/Page action from the previous section would lead to a 404 error in most VuFind instances, since that page is not defined as part of the core VuFind package. However, with the help of code generators, it would be easy to define such a page. Since controllers are just another plug-in (as described in chapter 17), some parts of the process should seem familiar by now. This section will show all of the steps involved.

For the purposes of this example, we will assume that you have a local code module called MyModule (see section 16.3.3 for more on local code modules) and a local theme called localtheme (see section 7.2 for more on theme generation). This example also assumes you are using VuFind 7.0 or later; earlier releases of VuFind did not support automatic generation of controllers, though of course the same effect could be achieved by manually creating files and editing configurations.

18.2.1 Creating the ExampleController
-------------------------------------

VuFind’s plugin generator can create controllers starting with VuFind 7.0, so this command will establish the ExampleController and configure it to be built using the standard VuFind\Controller\AbstractBaseFactory:

.. code-block:: console

   php $VUFIND_HOME/public/index.php generate plugin MyModule\\Controller\\ExampleController VuFind\\Controller\\AbstractBaseFactory

This command will create a file called $VUFIND_HOME/module/MyModule/src/MyModule/Controller/ExampleController.php. The class will be empty when generated, so you should edit it to add a basic pageAction(), like this:

.. code-block:: console
   
    <?php
     
    namespace MyModule\Controller;
     
    class ExampleController extends \VuFind\Controller\AbstractBase
    {
        /**
         *
         * @return \Laminas\View\Model\ViewModel
         */
        public function pageAction()
        {
            return $this->createViewModel();
        }
    }

All the pageAction does is return an empty view model, which will simply cause a view template to be rendered. And that leads to the next step…

18.2.2 Create the View Template
_______________________________

By convention, the template file displayed by the pageAction of the ExampleController should be named example/page.phtml, so you should create the file $VUFIND_HOME/themes/localtheme/templates/example/page.phtml. You can put any HTML and/or PHP logic in here that you like. For example:

.. code-block:: console 

   <p>Hello, world!</p>

At this point, if you add “Example/Page” to your VuFind home URL, you should see “Hello, world!” in your browser. This is because of the default route definition discussed in the previous section. However, there are some advantages to defining a specific route for this action, so there’s one more step to go…

18.2.3 Defining a Route
________________________

For simple routes like this one, VuFind has a “staticroute” generator you can call:

.. code-block:: console

   php $VUFIND_HOME/public/index.php generate staticroute Example/Page MyModule

This will create some route configuration inside MyModule’s config/module.config.php file defining a route named ‘example-page’ which matches an ‘/Example/Page’ URL and routes the user to the pageAction of the ExampleController. The main advantage to this over simply relying on the default route is that it allows you to generate links to your new page by using the URL view helper with the route name. For example, you could edit your $VUFIND_HOME/themes/localtheme/templates/example/page.phtml like this:

.. code-block:: console

   <?php $link = $this->url('example-page'); ?>
   <p>Hello, world! The link to this page is <a href="<?=$this->escapeHtmlAttr($link)?>"><?=$this->escapeHtml($link)?></a>.</p>
 
Obviously, this is a very simplistic example – but by understanding the relationship between routes, controllers and views, you can better understand VuFind’s code and behavior, and more easily extend its functionality to meet your local needs!

Additional Resources
--------------------

As noted earlier, the Laminas documentation provides more detail on controllers and routing; see https://docs.laminas.dev/laminas-mvc/ and https://docs.laminas.dev/laminas-router/ in particular.

The VuFind wiki page on controllers also contains some VuFind-specific details and up-to-date examples: https://vufind.org/wiki/development:plugins:controllers.

The tutorial video at https://vufind.org/wiki/videos:code_generators_2 includes an example of building a local custom controller.

Summary
-------

VuFind leverages the laminas-mvc system for model-view-controller architecture. The laminas-router component helps turn user request URLs into calls to action methods on controller classes. These methods process user input and help turn it into user output, either by generating HTTP responses or by delegating work to the view template system. Controllers are plug-ins, just like many other parts of VuFind, so they can be easily created and extended.

Review Questions
----------------

1.      How does the router impact which controller action gets called?
2.      Why would a controller action return an HTTP response object instead of a view model?
3.      What class do most VuFind controllers extend from?


