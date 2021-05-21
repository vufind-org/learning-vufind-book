Introduction to Laminas
***********************

Chapter 16. Introduction to Laminas
####################################


After reading this chapter, you will understand:

•       Why VuFind uses a framework.
•       How VuFind’s architecture is designed.
•       The importance of dependency injection.


16.1 Why Use a Framework?
-------------------------

To a new programmer, the biggest challenge of software development seems to be developing logic to solve problems. However, over time, an even bigger challenge is revealed: figuring out how to name and organize code so that it is easy to understand and maintain. One common way to reduce the burden of code organization is to pick a pre-existing framework that provides conventions and best practices for application design. Using a framework helps developers decide how to break down problems and arrange files, and it provides a common language across projects that share the same framework, making it easier to bring new developers on board.

When VuFind was first developed, it had a simple framework of sorts, but it was architecture built specifically for the development of VuFind. Over time, this original code became difficult to maintain; a lot of shared logic was copied-and-pasted between files rather than abstracted in a more reusable way, and most of the application’s files were dumped into a small number of directories, making it hard to find things. When the time came to develop a 2.0 release, rather than try to “fix” the existing system, the VuFind community decided to try rebuilding the application on an established open source framework, both to take advantage of the aforementioned advantages of a framework and to bring the software more in line with best practices of the time. While this proved to be a complex and laborious undertaking, it paid off in the long run by improving code quality and making subsequent extensions and customizations much easier to develop.

Several possible frameworks were considered for VuFind, but ultimately Zend Framework 2 was chosen, both because it was cutting-edge at the time (taking full advantage of then-new PHP features like namespaces) and because it provided very strong support for overriding core functionality, theming, and adding plug-ins. Zend Framework 2 never became the most popular PHP framework broadly, in large part due to its high complexity compared to some of its competitors, but its complexity is exactly what provided the flexibility needed to make it a perfect fit for VuFind.

Over the years since VuFind first rebuilt on Zend Framework 2, the PHP community has moved forward and things have changed. The once-monolithic Zend Framework 2 evolved into Zend Framework 3, which was split into smaller, more independently-reusable components as the Composer dependency management tool gained popularity and made it easier to mix and match third-party PHP code. Eventually, Zend Framework rebranded itself as Laminas. As of VuFind 7.0, all references to “Zend” in the VuFind code have been replaced with equivalent Laminas code, and some Zend components have been replaced with better tools from other developers. However, in spite of the many things that have changed over the years, the basic ideas behind Zend Framework 2 are still a very important part of VuFind and have supported its ongoing success.

While migrating VuFind to a framework was a significant challenge, once the migration was complete and the framework’s basic conventions were understood, customizing and extending the software became quite simple. The VuFind team has done the hard work, so now end users can reap the benefits. This chapter provides background on the key concepts needed to understand and take advantage of VuFind’s design. It assumes a basic understanding of object-oriented programming with PHP; exploring those fundamentals is beyond the scope of this book, but many other books and online tutorials exist to fill that gap (see Additional Resources below for some suggestions).

16.2 Model-View-Controller Architecture (laminas-mvc)
-----------------------------------------------------

VuFind uses a model-view-controller architecture. The idea of this architecture is to separate code into three distinct parts: models, which represent data; views, which are used to display information to the end user; and controllers, which implement the “business logic” that makes the application actually do things.

This is a useful separation, because it allows key aspects of the application to be developed, tested and changed independently of one another. For example, best practices for web design change at a fairly rapid pace; by separating view logic from other parts of the application, it is possible to periodically refresh modernize VuFind’s interface by changing all of its views, without having to worry about any of the code in the models or controllers. Similarly, the model/controller distinction separates code that deals with HOW to do things, from code about WHAT to do, which tends to improve the clarity of the code.

Of course, “model-view-controller” is a theoretical ideal, and VuFind is a real-world application, so every piece of code in VuFind does not necessarily fit neatly into a “model,” “view,” or “controller” bucket. In particular, the line between models and controllers can blur fairly easily. However, the most important idea here is “separation of concerns,” and VuFind’s design aims to split code into logical (and, in most cases, overridable) chunks that work together well but provide clear boundaries between one another.

VuFind’s specific implementation of model-view-controller uses the Laminas implementation of the architecture (laminas-mvc), with a few VuFind-specific customizations to add better support for themes. The views are all located in the $VUFIND_HOME/themes directory, as discussed in chapter 7. The models and controllers are all defined in PHP classes within VuFind’s code, and all are set up using configuration files in VuFind’s modules, leveraging the Laminas service manager. Both modules and the service manager are discussed in more detail below. Controllers will be given more attention in chapter 18.

16.3 Core Modules (laminas-modulemanager)
----------------------------------------- 

One of the attractive features of Laminas (and Zend Framework before it) is the ability to organize code into modules. A Laminas module consists of a directory containing code and associated configuration. Several modules can be composed together to build an application.

16.3.1 Why Use Modules?
_______________________

In general terms, it is almost always beneficial to organize code into logical units, where closely related code is grouped together and unrelated code is separated. Laminas modules provide one mechanism for this type of organization.

From VuFind’s perspective, though, the most exciting feature of Laminas modules is the way they allow merging of configuration. When you load modules into an application in a particular order, the configuration settings from later modules are merged with, or fully override, settings from earlier modules. This is ideal for an application like VuFind, where there is a core set of defaults, but users may wish to program their own changes. In VuFind, users define all of their local code and application-level configuration within their own module, and this module is loaded last so that it can override VuFind’s defaults. This encourages developers to program customizations in a way that clearly expresses how their functionality differs from the core, and to change only the pieces that really need changing. This approach can make long-term maintenance easier, since a new developer taking over a project can likely make more sense of a discrete custom code module than they could a collection of minor edits hidden within the core code of the project.

Code modules are also interesting because they can be turned on or off conditionally within the code of the application. VuFind takes advantage of this in a couple of ways. First of all, some modules, like the administration panel, are only loaded when certain configuration settings are turned on. This reduces the odds of sensitive functionality being accessed by accident – if the code is completely inaccessible to the application, there are fewer potential attack vectors against it. Additionally, since modules can be loaded based on configuration, this provides part of the foundation of VuFind’s “multi-tenant” functionality, where a single installation of the software can serve up differently-configured instances based on the incoming URL.

Code modules are also a way of sharing functionality between Laminas-based applications; for example, VuFind makes use of a third-party module for integrating the Whoops error handler, a useful tool for debugging run-time errors.

If you are interested in seeing exactly how VuFind loads its modules, take a look at the $VUFIND_HOME/config/application.config.php file, which contains most of the high-level Laminas application setup logic.

16.3.2 VuFind’s Built-In Modules
________________________________

VuFind contains several code modules, found under the $VUFIND_HOME/modules directory; as of release 7.0, they are:

•       VuFind – the core of VuFind, containing the majority of the code.
•       VuFindAdmin – code for VuFind’s web-based administration panel.
•       VuFindApi – code for exposing VuFind search results and records through an API.
•       VuFindConsole – code for all of VuFind’s command-line utilities.
•       VuFindDevTools – code for utilities used by VuFind developers (only active when development mode is turned on in $VUFIND_LOCAL_DIR/httpd-vufind.conf).
•       VuFindSearch – code for VuFind’s search system (including low-level backend code).
•       VuFindTheme – code for VuFind’s theme system.


There is also a module called VuFindLocalTemplate, which is not used directly but is the basis for building local modules, as described in the next section.

16.3.3 Local Custom Modules
___________________________

Most VuFind installations have at most one local code module; many have none at all. If you install VuFind using the Debian package as described in section 2.2, no module will be set up by default. Fortunately, adding a module is fairly straightforward and can be done either manually or automatically.

To do it manually, just copy the $VUFIND_HOME/VuFindLocalTemplate directory and its contents to a new directory name (corresponding with the desired name of your local code module). Edit the namespace declarations in both PHP files (Module.php and config/module.config.php) within your new module to match the name of the module. Then, edit $VUFIND_LOCAL_DIR/httpd-vufind.conf, uncomment the VUFIND_LOCAL_MODULES environment variable line, and insert the appropriate module name. Finally, restart Apache so the change takes effect.

To do it automatically, run php *$VUFIND_HOME/install.php –* the VuFind install script asks a question about local modules and automatically copies and configures VuFindLocalTemplate for you based on your answer. Just be sure you answer the other questions in the script consistently with your current configuration so that you don’t break anything. If something goes wrong, don’t worry – the install script backs up your $VUFIND_LOCAL_DIR/httpd-vufind.conf to a filename with a timestamped extension, so you can copy it back to fix any problems.

Also note that, once you are using a local module, you will also want to export its name into the $VUFIND_LOCAL_MODULES environment variable whenever you are working with VuFind’s command-line tools to be sure your custom code is properly accounted for. If you installed VuFind using standard practices, you can most easily ensure this variable is set correctly by adding a line to the /etc/profile.d/vufind.sh file where $VUFIND_HOME and $VUFIND_LOCAL_DIR should already be getting set up. Exporting the local module name is especially important when you are working with code generators, since they need to be able to find your module in order to add code to it; see section 9.3 for an example of code generators in action.

16.4 Dependency Injection and Containers (laminas-servicemanager)
-----------------------------------------------------------------

As noted earlier, deciding how to break an application into parts, and where to put those parts, is a big challenge. Another related challenge is figuring out how to let those parts interact with one another. It is very common for software components to have dependencies on one another, but if these relationships are not managed carefully, code can become hard to maintain and customize. To address this particular problem, VuFind follows Laminas’ lead and uses dependency injection.

The idea of dependency injection is simple: every software component should declare up front what other components it depends upon for its functionality. When the component is constructed, those dependencies should be injected into it. For example, VuFind contains code that talks to Solr over HTTP. In order to do this, it needs access to code that speaks the HTTP protocol. When we build the Solr connector, we inject the HTTP client so it has access to the functionality that it needs.

This leads to a sort of “inside out” feeling to the code – you have to build the more specific, low-level components first in order to pass them along to the higher-level components that depend upon them. But consider the alternative: we could have each component essentially build its own dependencies as it needs them. For example, rather than injecting an HTTP client into the Solr connector, we could have the Solr connector build its own HTTP client at the time that it needs to use it. However, the dependency injection approach offers some important advantages.

First of all, if we want to share components, dependency injection is the way to go. If the Solr connector builds its own HTTP client every time it runs, that may be less efficient than having a single HTTP client object that is shared between multiple components in the system. Secondly, dependency injection makes customization easier. If we want to extend the HTTP client to add some of our own custom functionality to it, we can inject that custom HTTP client into a Solr connector without changing the Solr connector (as long as we maintain compatibility with the “standard” client). However, if the Solr connector built its own HTTP client internally, we would have to customize not just the HTTP client code, but also the Solr connector code, in order to change things. Finally, dependency injection makes it easier to test code. Writing automated tests is an important part of software development, ensuring that code works correctly when it is first written and continues to work correctly even while other parts of the system change. With dependency injection, it is possible to inject a “test double” that simulates a dependency; in the Solr connector example, this could allow us to test the Solr connector code without actually having to have a running instance of Solr – we could just inject a test double that returns pre-formatted responses that simulate a real Solr server.

Of course, despite its many advantages, dependency injection has one obvious disadvantage: it can make the process of building objects in code more complicated, because components need to be assembled and injected into one another like nesting dolls. Doing all of this by hand could quickly become unwieldy. Fortunately, a very useful Laminas component, the ServiceManager, comes to save the day here.

The Laminas ServiceManager is a class whose responsibility is building and storing what it calls “services,” which in practice are almost always PHP objects. It acts as a configurable registry: configuration specifies the services contained within the manager along with instructions on how to build those services, and when a service is requested, the service manager builds it (if it hasn’t already been built) and then returns it to the calling code. The service manager can be configured to build a brand new object on every request, but by default, it only builds objects once and then recycles them, making it a useful tool for sharing components with one another.

To make the most of the service manager, the application utilizing it needs to be programmed so that, instead of directly building objects, it always fetches them from the service manager. By introducing this extra layer of abstraction, we also gain a lot of power and flexibility. If we want to override a particular service, we can simply reconfigure the service manager to use our customized code instead of the normal service; we don’t have to change anything else, and we can be confident that our custom code will be used everywhere that the original code was used. To contrast, if an application used inline construction of objects instead of the service manager, replacing a frequently-used class might require changes to dozens or even hundreds of files and would be a source of maintenance headaches every time a new version of the software was released.

In addition to the power of abstraction, the service manager helps with dependency injection in a very important way. The service manager can build objects using a variety of strategies, but the most common (and the one used by most of VuFind’s code) is with the help of factory classes. As the name suggests, a factory is a class used for building other objects. The factories used by the Laminas service manager are powerful for two reasons: they can use the service manager itself as part of the process of building objects (greatly simplifying the job of managing nested dependencies), and they are reusable: as long as you adhere to certain conventions, like ensuring that service names in the service manager always correspond to PHP class names, the same factory can be used to build objects belonging to multiple classes. Reusability can be a real time saver when many objects in your application have the exact same set of dependencies.

All of this should become more clear with an example – let’s implement the “Solr connector with an HTTP client dependency” example with Laminas\ServiceManager. We’ll assume for the sake of example that the HTTP client has no other dependencies, but the Solr connector needs the HTTP client. To do this, we would configure the ServiceManager with two services. First, the HTTP client could be configured to use the ServiceManager’s generic “InvokableFactory” – a reusable factory which builds an object in the simplest possible way, without passing any dependencies into it. Next, the Solr connector could be configured to use a new custom factory. In the Solr connector’s factory code, we could fetch the HTTP client from the service manager 

1.      The application asks the service manager to provide the Solr connector.
2.      The service manager looks up how to build a Solr connector, and calls the appropriate factory class.
3.      The Solr connector factory asks the service manager to provide the HTTP client.
4.      The service manager looks up how to build the HTTP client, and calls the Invokable factory, which immediately builds the HTTP client object.
5.      The Solr connector factory now builds the Solr connector using the HTTP client, then returns the result to the application.

For a very complex object, the service manager might end up triggering a complex chain reaction, using many factories and looking up many additional services… but as long as each factory simply asks the service manager for the immediate dependencies of the object being built, the developer doesn’t need to worry too much about those implementation details. The chain may be complex, but each link in the chain is easy to build and clear to read.

Understanding the role of the service manager is probably the single most important key to understanding VuFind’s code. It is not necessarily obvious at first, but the power of this one tool informs much of VuFind’s design, both in how the code is broken down into parts and in how nearly everything can be customized and extended. If this is not yet clear, you may wish to revisit this chapter after you have had more experience working with the code. Chapter 17, discussing plug-ins, may shed additional light as well.

Additional Resources
---------------------

For a basic introduction to PHP programming, O’Reilly’s Programming PHP by Tatroe and McIntyre (in its 4th edition as of this writing) covers all of the fundamentals. For a free online alternative, the w3schools tutorial explains much of the same material: https://www.w3schools.com/php/.

The Wikipedia page on Model-View-Controller architecture provides some broader context and analysis on that design pattern: https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller.

You can learn more about the ecosystem of Laminas components at the project’s official website: https://getlaminas.org/.

The Wikipedia page on Dependency Injection goes into greater depth on the concept: https://en.wikipedia.org/wiki/Dependency_injection.

VuFind’s Developer Manual contains more specific notes on architecture and best practices: https://vufind.org/wiki/development.

Summary
-------

As an open source project designed to serve many users and many use cases, it is important for VuFind to follow shared best practices and to structure its code in a way that is easily understood and modified. The project uses Laminas components as its basis in order to meet these goals. Specifically, it uses the Laminas model-view-controller architecture to separate data, business logic and presentation templates, it uses the module manager to group related code together and to support configuration overrides, and it uses the service manager to allow easy class overriding and factory-based dependency injection. All of these tools work together to inform the shape of new code as the project grows and to provide easy hooks for developers to use while customizing local instances.

Review Questions
----------------
1.      What is the difference between a model, a view, and a controller?
2.      What is one disadvantage of dependency injection, and how does VuFind mitigate it?
3.      Name three Laminas components used by VuFind, and explain their significance.

