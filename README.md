Buildout
========

In this document, we will go through the basic steps to configure zc.buildout for a (very) simple python project. It aims to break the process down to its most simple form.

Laying out a basic project
--------------------------

We will write a simple python program that takes a search query on the command line, and prints the names of the first 10 search results (first page) found when submitting it to google. The implementation will use 2 dependancies that aren't installed in a default python setup, so we will have the opportunity to install those during the buildout process.

Make sure you have a folder set up to mess around in, while you're reading this document. For the purposes of this guide, let's assume the folder is ~/src/zc\_sandbox

**Add a subfolder, called "googler", to ~/src/zc\_sandbox.** This will contain the actual code

**_Add an empty \_\_init\_\_.py file to this directory.** This will make it a valid python module

**Add a file called googler.py as well,** containing the following code:

    from sys import argv

    import requests
    from BeautifulSoup import BeautifulSoup


    def _get_search_page(query):
        url = 'http://www.google.com/#q=buildout'
        result = requests.get(url)
        if result.status_code not in [500, 404]:
            return result.content


    class Link(object):
        def __init__(self, url, name):
            self.url = url
            self.name = name

        def __str__(self):
            return u'<{0}> {1}'.format(self.name, self.url).encode('utf-8')

    class GoogleSearch(object):
        def __init__(self, query):
            self.htmlresults = ''
            url = 'http://www.google.com/search?q={0}'.format(query)
            result = requests.get(url)
            if result.status_code not in [500, 404]:
                self.htmlresults = result.content
            self._parse()

        def _parse(self):
            self.results = []
            raw_links = []
            soup = BeautifulSoup(self.htmlresults)
            results_parents = soup.findAll(attrs={'class': 'r'})
            for parent in results_parents:
                raw_links.extend(parent.findAll('a'))

            self.results = [Link(raw.attrMap['href'], raw.text) for raw in raw_links]

    if __name__ == '__main__':
        for result in GoogleSearch(argv[1]).results:
            print result.name

Adding a setup.py
-----------------
Create a file called setup.py in ~/src/zc\_sandbox, with the following code:

    from distutils.core import setup


    setup(
            name='Googler',
            version='0.0.1dev',
            author='Yourname mcCoder',
            packages=['googler']
    )

Adding a buildout.cfg
---------------------
Next, we will want to create a buildout configuration. In ~/src/zc\_sandbox, add a file called buildout.cfg, containing the following:

    [buildout]
    develop = .
    parts = 
            googler
            py

    [googler]
    recipe = zc.recipe.egg
    eggs = 
            googler
            requests
            BeautifulSoup

    [py]
    recipe = zc.recipe.egg
    interpreter = py
    eggs = 
            ${googler:eggs}

Bootstrapping
-------------
Next, we will use the standard bootstrapping module. We will download the module, and run it to initialize the standard directory structure. Run the following commands in ~/src/zc\_sandbox:

    wget http://svn.zope.org/\*checkout\*/zc.buildout/trunk/bootstrap/bootstrap.py
    python bootstrap.py

_Note:_ In my case, the latter command produced a "pkg_resources.VersionConflict", but I've found this can be avoided using:

    python bootstrap.py -d -v 2.1.1

instead

This should leave you with the following directory structure:

    ~/src/zc_sandbox
    |-- bin
    |   `-- buildout
    |-- bootstrap.py
    |-- buildout.cfg
    |-- develop-eggs
    |-- eggs
    |   |-- distribute-0.6.49-py2.7.egg
    |   `-- zc.buildout-2.1.1-py2.7.egg
    |       |-- EGG-INFO
    |       |   |-- dependency_links.txt
    |       |   |-- entry_points.txt
    |       |   |-- namespace_packages.txt
    |       |   |-- not-zip-safe
    |       |   |-- PKG-INFO
    |       |   |-- requires.txt
    |       |   |-- SOURCES.txt
    |       |   `-- top_level.txt
    |       |-- README.rst
    |       `-- zc
    |           |-- buildout
    |           |   |-- allowhosts.txt
    |           |   |-- bootstrap_cl_settings.test
    |           |   |-- bootstrap.txt
    |           |   |-- buildout.py
    |           |   |-- buildout.pyc
    |           |   |-- buildout.txt
    |           |   |-- configparser.py
    |           |   |-- configparser.pyc
    |           |   |-- configparser.test
    |           |   |-- debugging.txt
    |           |   |-- dependencylinks.txt
    |           |   |-- downloadcache.txt
    |           |   |-- download.py
    |           |   |-- download.pyc
    |           |   |-- download.txt
    |           |   |-- easy_install.py
    |           |   |-- easy_install.pyc
    |           |   |-- easy_install.txt
    |           |   |-- extends-cache.txt
    |           |   |-- __init__.py
    |           |   |-- __init__.pyc
    |           |   |-- meta-recipes.txt
    |           |   |-- repeatable.txt
    |           |   |-- rmtree.py
    |           |   |-- rmtree.pyc
    |           |   |-- runsetup.txt
    |           |   |-- setup.txt
    |           |   |-- testing_bugfix.txt
    |           |   |-- testing.py
    |           |   |-- testing.pyc
    |           |   |-- testing.txt
    |           |   |-- testrecipes.py
    |           |   |-- testrecipes.pyc
    |           |   |-- tests.py
    |           |   |-- tests.pyc
    |           |   |-- update.txt
    |           |   `-- windows.txt
    |           |-- __init__.py
    |           `-- __init__.pyc
    |-- googler
    |   |-- googler.py
    |   `-- __init__.py
    |-- parts
    `-- setup.py

Among a lot of other things, the bootstrapper added a bin/buildout script. This script should install the googler program, and both dependencies. Simply run:

    bin/buildout
