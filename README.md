# Learning VuFind

This book provides basic insights for installation, configuration and customisation of the Open Source Discovery Software VuFind.

For more information on VuFind please visit: https://www.vufind.org

## Build documentation

For locally building the documentation you need the Sphinx documentation generator: https://www.sphinx-doc.org

Please refer to the official Sphinx documentation on how to install on your system.

After having installed Sphinx on your system first clone the `learning-vufind-book` into your workdingdir

    $ git clone https://github.com/vufind-org/learning-vufind-book.git 

change directory

    $ cd learning-vufind-book

and build the Learning VuFind Book

    $ make html

When build process is finished you will find the built html-files in `./build/html`. To view the built documentation open `./build/html/index.html`.

If you prefer a PDF version, you can instead run

    $ make latexpdf

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>.
