Vufind Basics
*************

Chapter 1. Introduction to VuFind
#################################

Learning Objectives
-------------------

After reading this chapter, you will understand:

1. Why the VuFind project was created.
2. Key features of the VuFind software.
3. Common reasons for using VuFind.

1.1 What is (and isn’t) VuFind?
--------------------------------

1.1.1 History
____________

In 2007, libraries faced a problem: their mission was to help their users find information, but the information landscape and user expectations were changing. The Google search engine had already been around for a decade, and its single search box approach was becoming ubiquitous. At the same time, many common library search applications were still built around assumptions inherited from the days of card catalogs and featured user interfaces designed during the very earliest days of the World Wide Web. There was a strong need to provide a more user-friendly search experience for library resources.

A group of software developers and designers at Villanova University decided to address this problem through the creation of a search application built using commonly-available, free, open source components. A prototype was built, and this sparked interest from other libraries around the world. A community of software developers quickly grew around the software, collaboratively growing and improving it. After a few years of experimentation, the first stable release was unveiled in early 2010, and enhanced versions have been released every year since then. While members have come and gone, VuFind’s community still exists today, continuing to develop, update and improve the software.

1.1.2 Features
______________

VuFind’s primary purpose is to provide a user-friendly search experience for diverse resources. The most common application of VuFind is as a library catalog, but it is also frequently used to search collections of journal articles, archival holdings, and even websites. VuFind offers options for searching locally-managed records and also for connecting to third-party search services, such as OCLC’s WorldCat union catalog, and many commercial “web-scale discovery” services. In cases where many different types of resources are available for searching, VuFind offers multiple options for combining and organizing search results. The common denominator is that, wherever the records come from, VuFind presents them in a consistent user interface, reducing user confusion and centralizing the search experience.

In addition to providing robust search functionality, VuFind is also designed to provide easy access to library users’ accounts. Users can renew books, place holds, and view their fines. As with search, VuFind is designed to provide a simple and consistent user experience; while many different systems may be used to manage this information behind the scenes, VuFind provides a single interface for public consumption. This can be especially valuable for institutions planning on changing their back-end software systems; once VuFind is established as the user interface, changes can be made “behind the scenes” without users noticing any significant difference.

The VuFind software is designed to be extremely configurable and customizable. In addition to interoperating with a wide range of other systems, most of its behaviors can be adjusted through the editing of text-based configuration files. While it contains a wide range of features, most can be turned on or off to suit local needs. The high level of configurability is designed to minimize the need to write local code, but in cases where deep customization is needed, the software’s architecture is designed to make it easy to override components and add functionality by building small plugin-ins.

1.1.3 Limitations
_________________

While VuFind is extremely powerful and flexible, it is important to understand its scope: it is designed as a user interface layer on top of other systems. It provides search capability for existing records, and it provides friendly access to existing user accounts. It is not designed to create or maintain records, however, nor is it meant to be your primary user management system. For library applications, the expectation is that you have an existing integrated library system or library services platform for managing resources and accounts. For other applications, it is assumed that the records you are indexing are created and stored elsewhere.

These limitations are intentional: managing records and searching records have different requirements. Systems that attempt to do everything usually have to make compromises in terms of performance or functionality. By separating management tasks from user-facing tasks, you can have a more efficient overall system. VuFind is designed for efficient searching, and it allows other systems to focus on other priorities, like maintaining data integrity and providing record editing interfaces.

1.1.4 Use Cases
_______________

There are many possible reasons for using VuFind; here are a few common situations:

•       You have multiple sets of resources (e.g. a library catalog, an institutional repository and a website), and you would like to make them searchable through a single box.
•       You subscribe to a commercial service (such as a library services platform or a web-scale discovery service), but you want more local control over the user experience than the vendor-provided software offers.
•       You want to provide a user interface that will remain stable over time, even if you replace some components of your infrastructure (such as migrating from one integrated library system to another).
•       You have a locally-developed information resource (such as a bibliography or archival collection), and you want to provide a public user interface for the data.

Each of these goals can be accomplished with VuFind using the appropriate configuration options. This text will give you the tools necessary to solve these problems, and many more.

1.2 Prerequisites for Using and Understanding VuFind
----------------------------------------------------

In order to make the most of this book, it is helpful to have some background knowledge in a few areas. While going into depth introducing these topics is beyond the scope of this book, many online tutorials and textbooks are available on each of these subjects, should you wish to familiarize yourself with them before proceeding. 

While VuFind can run on a variety of operating systems (see section 2.1 for more details), running it on Linux is the preferred option, and so a basic understanding of the Linux command line and file system will be extremely helpful. Because most of VuFind’s configuration is accomplished by editing text files, comfort with a text editing tool of some sort – such as nano or vi – is a must; fortunately, a wide variety of user-friendly tools are available.

If you plan to customize VuFind’s look and feel – and most VuFind users will at least want to change some logos and adjust some colors – you should have a basic understanding of web technologies like HTML and CSS. For more advanced customization, an understanding of Javascript programming and CSS pre-processors like LESS or SCSS may be helpful as well.

For advanced customization of VuFind’s behavior, or for integrating the software with systems that are not already supported, you will need an understanding of the PHP programming language and object-oriented programming. Some of this book’s later chapters (part 6) will assume these basics, though they will go into some depth explaining the Laminas framework that provides structure to VuFind’s software components. You can accomplish a great deal with VuFind without having to do any programming, so you can freely skip these chapters if you are not comfortable with this level of detail; however, if you want to have full control over the application, this background knowledge will prove extremely valuable.

1.2.1 VuFind Community
______________________

One of the advantages of using open source software is that successful applications are supported by a community of users and developers who can often be a valuable resource. VuFind is no exception; it has an active and supportive community which provides several options for communication. Documentation for the software is provided through a searchable wiki (https://vufind.org/wiki). When the documentation does not answer a question, users can ask questions on multiple mailing lists or via a Slack community (see the “Support” page of https://vufind.org for the most up-to-date links). The community also streams a regular, free online Community Call to coordinate development of the software, provide updates on new feature, and answer questions; the schedule for this can also be found on the website, and all are welcome to join in.

Summary
-------

VuFind is an open source project designed to give libraries (and other cultural heritage institutions) more control over their web-based search and account management experience. Users with a basic understanding of Linux commands and HTML/CSS have a great deal of power to configure and customize the software; PHP programmers can go even further. This book will help provide a roadmap to VuFind’s features and options; the project’s friendly community can help answer questions when they arise.

Review Questions
----------------

1. What are some of VuFind’s core features?
2. What are some of VuFind’s limitations, and why do they exist?
3. Where can you go to get help with VuFind?
4. What technologies should you familiarize yourself with to make the most of VuFind?
5. Why might a library wish to install VuFind?

