git-fat
=======

A tool for managing large binary files in git repositories.

Introduction
------------

Git-fat is a tool written by `jedbrown <https://github.com/jedbrown/git-fat>`_.
This repository / pypi package is a fork which we are
`actively trying to resolve <https://github.com/jedbrown/git-fat/pull/19>`_.
With that said, the repository placeholder format is compatible with both, so
they should be interchangeable for now.  Please take care to check which one
you are using before opening issues on either repository, and include as much
information as possible so that we are able to help you as best we can.

Now an explanation about what (either) ``git-fat`` does:

Checking large binary files into a distributed version control system is
a bad idea because repository size quickly becomes unmanageable. Numerous
operations take longer to complete and fresh clones become something
that you start and wait for a bit before coming back to them.
Using ``git-fat`` allows you to separate the storage of large-files from
the source while still having them in the working directory for your project.

Features
--------

-  Cloning the source code remains fast because binaries are not
   included
-  Binary files really exist in your working directory and are not
   soft-links
-  Only depends on Python 2.7 and a backend
-  Supports anonymous downloads of files over http

Compatibility
-------------

OS support: Linux, Mac, Windows.

Python support: only 2.x

Installation
------------

You can install ``git-fat`` using pip.

::

    pip install git-fat

Or you can install it simply by placing it on your path.

::

    curl https://raw.github.com/cyaninc/git-fat/master/git_fat/git_fat.py \
    | sudo tee /usr/local/bin/git-fat && sudo chmod +x /usr/local/bin/git-fat

Installation on Windows
-----------------------

To install from PyPI type:

::

    pip install --upgrade git-fat

It is required for the Python27/Scripts directory to be in PATH.

See also other scripts in the win32/ directory:

-  install_win.py - this script ensures that a binary wheel package (.whl)
   is installed from PyPI. If for example there is a Linux source package
   (.tar.gz) available and its version number is higher than any of the
   available .whl packages, then the pip tool would install the Linux
   source package and for obvious reasons it wouldn't work.
-  run_tests_xxx.bat - run unit tests
-  setup.bat - install package from sources
-  setup_wheel.bat - create a binary Python Wheel package in the dist/
   directory and install that package

Configuration
-------------
``git-fat`` is configured by defining in the ``.gitattributes``
file at the root the repository which files it should manage and by
providing git-fat-specific configuration settings, typically  in a
``.gitfat`` file also at the root of the repository.

Note that version 0.4.0 of ``git-fat`` introduces new optional constructs
in the configuration domain without sacrificing support for the configuration
syntax used in previous versions (called legacy configuration files
below).

Modes of operation
~~~~~~~~~~~~~~~~~~

When it comes to the git-fat-specific configuration settings, there is
two modes of operation.

Implicit configuration (default)
''''''''''''''''''''''''''''''''

When no configuration files are passed as arguments to the ``-c`` option,
``git-fat`` parses the following configuration files in the order they
are listed here:

1. If ``$GIT_FAT_CONFIG`` is defined and contains a colon-separated list of files,
   ``git-fat`` parses those files, overriding previously defined settings with the
   same name if they exist.
2. The regular Git configuration files (see the ``FILES`` section of  ``man git-config``)
   are parsed, overriding any previously defined configuration settings with the same name
   if they exist.
3. The ``.gitfat`` configuration file at the root of the repository is parsed,
   overriding previously defined configuration settings with the same name if they exist.
   In this case the ``.gitfat`` file is considered to be the *main configuration file*.

Explicit configuration
''''''''''''''''''''''

When one or more configuration files are passed as arguments to the ``-c`` option,
``git-fat`` processes each of those files in the order they were passed,
overriding previously defined configuration settings with the same name if they exist.

In this case, the last one of the files is considered to be the *main configuration file*.

Interpolation
~~~~~~~~~~~~~

``git-fat`` performs interpolation on the setting values that are used by ``git-fat`` itself.
Interpolation consists on replacing *setting names* delimited by curly braces {} with the value
of those settings. *Setting names* are dot-separated strings with the section, subsection (if
applicable) and key names of settings defined in the parsed configuration files.

For the sake of integrity, it is possible to specify default values for interpolation
in the *main configuration file* (and only there).

::

    [defaults "original-section"]
    key1 = default-value-1

When placed in the *main configurtion file*, the section above means that if -and only if- a
setting with *setting name* ``original-section.key`` has not been defined in any configuration
file, the interpolation of the string ``{original-section.key}`` will yield ``default-value-1``,
otherwise it will yield the value it was given in the configuration file where it was defined.

Example files
~~~~~~~~~~~~~~

``.gitattributes``
'''''''''''''''''''
This file is located at the root of the repository and determines which files
get converted to ``git-fat`` files. See
`git attributes <http://git-scm.com/book/en/Customizing-Git-Git-Attributes>`_
for further information.

::

    cat >> .gitattributes <<EOF
    *.deb filter=kat -crlf
    *.gz filter=fat -crlf
    *.zip filter=fat -crlf
    EOF

``.gitfat`` (legacy)
''''''''''''''''''''''''

``.gitfat`` is typically located at root of the repository but could be
explicitly passed as argument to the ``-c`` options (see *Modes of operation* above).

::

    [rsync]
    remote = storage.example.com:/path/to/store
    user = git
    port = 2222
    [http]
    remote = http://storage.example.com/store

In this case the first section of the file is treated as the default backend.

For legacy ``.gitfat`` files indentation is discouraged as it is not supported by
previous versions of ``git-fat``.

``.gitfat`` (namespaced)
''''''''''''''''''''''''''''''''''''''

::

	[gitfat]
		backend = rsync
		canned-error-message = "A message to append on stderr on run-time errors"
	[gitfat "rsync"]
		remote = storage.example.com:/path/to/store
		user = git
		port = 2222
	[gitfat "http"]
		remote = http://storage.example.com/store

In this case the default backend is explicitly mentioned.

Note that indentation is possible, but it must be done with tabs.

This syntax is not compatible with ``git-fat`` versions prior to 0.4.0.

``.gitfat`` (interpolated)
''''''''''''''''''''''''''''
This example meant to be used in combination with *supporting
configuration files* (see below) and makes uses of the interpolation
mechanism described above.

::

	[gitfat]
		canned-error-message = "A message to append on stderr on run-time errors"
	[gitfat "rsync"]
		remote = {siteconfig.synchost}:{siteconfig.syncroot}
		user = git
		port = 2222
	[gitfat "http"]
		remote = {siteconfig.http-url}
	[defaults "gitfat"]
		backend = rsync
	[defaults "siteconfig"]
		synchost = localhost
		syncroot = /path/to/local/store/mount
		http-url = http://storage.example.com/store

Note that indentation is possible, but it must be done with tabs.

This syntax is not compatible with ``git-fat`` versions prior to 0.4.0.

*Supporting configuration file*
'''''''''''''''''''''''''''''''
*Supporting configuration files* are typically used to define settings that require
site, repository or user variability.

Examples of supporting configuration files are:

* An arbitrary file path contained in ``$GIT_FAT_CONFIG``.
* ``~/.gitconfig`` for user-specific settings.
* ``.git/config`` at the root of a repository for local, repository-wide settings.
* An arbitrary file path passed to ``git-fat`` as an argument to ``-c`` option not
  being the last ``-c`` option in the invocation call.

A configuration file thought to complement the interpolated ``.git-fat`` file above
may look like the following:

::

	[gitfat]
		backend = rsync
	[siteconfig]
		synchost = storage.example.com
		syncroot = /path/to/store
		http-url = http://storage.example.com/store

This syntax is not compatible with ``git-fat`` versions prior to 0.4.0.

Usage
-----

The commands described below require that the ``.gitattributes`` and
the configuration files (typically just ``.gitfat``) have been set up
as described in the *Configuration* section above.  Remember to commit
the ``.gitfat`` and ``.gitattributes`` files so that others will be
able to use them.

The command below is used to initialize the repository. This adds a line to
``.git/config`` telling git what command to run for the ``fat`` filter referred to in
the ``.gitattributes`` file.

::

    git fat init

Now when you add a file that matches a pattern in the ``.gitattributes``
file, it will be converted to a fat placeholder file before getting
committed to the repository. After you've added a file **remember to push
it to the fat store**, otherwise people won't get the binary file when
they try to pull fat-files.

::

    git fat push

After we've done a new clone of a repository using ``git-fat``, to get
the additional files we do a fat pull.  This will pull the default backend
which can be explicitely mentioned as in the namespaced ``.gitfat`` example
above, or else is determined by the first entry in the *main configuration
file*, as in the legacy ``.gitfat`` example above.

::

    git fat pull

To specify which backend to use when pulling or pushing files, then simply
list the backend type after the pull or push command.

::

    git fat pull http

To list the files managed by ``git-fat``

::

    git fat list

To get a summary of the orphan and stale files in the repository

::

    git fat status

Orphans are files that exist as placeholders in the working copy. Stale
files are files that are in the ``.git/fat/objects`` directory, but have
no working copy associated with them (e.g. old versions of files).

To find files over a certain size, use git fat find. This example finds
all objects greater than 10MB in git's database and prints them out.

::

    git fat find 10485760

Implementation notes
--------------------

For many commands, ``git-fat`` by default only checks the current
``HEAD`` for placeholder files to clone. This can save on bandwidth for
frequently changing large files and also saves on processing time for
very large repositories. To force commands to search the entire history
for placeholders and pull all files, call ``git-fat`` with ``-a``. e.g.

::

    git fat -a pull

If you add ``git-fat`` to an existing repository, the default behavior
is to not convert existing binary files to ``git-fat``. Converting a
file that already exists in the history for git would not save any
space. Once the file is changed or renamed, it will then be added to the
fat store.

To setup an http server to accept ``git-fat`` requests, just configure a
webserver to have a url serve up the ``git-fat`` directory on the
server, and point the ``.gitfat`` http remote to that url.

Retroactive Import
------------------

You can retroactively import a repository to ``git-fat`` using a combination
of ``find`` and ``index-filter`` used with git's ``filter-branch`` command.

Before you do this, make sure you understand the consequences of
`rewriting history <http://git-scm.com/book/ch6-4.html>`_ and be sure to
backup your repository before starting.

First, clone the repository and find all the large files with the
``git fat find`` command.

::

    darthurdent at betelgeuse in /tmp/git-fat-demo (master)
    $ git fat find 5123123
    761a63bf287867da92eb420fca515363c4b02ad1 9437184 flowerpot.tar.gz
    6c5d4031e03408e34ae476c5053ee497a91ac37b 10485760 whale.tar.gz


Review the files and make sure that they're what you want to exclude from the
repository.  If the list looks good, put the file names into another file that
will be read from during ``filter-branch``.

::

    darthurdent at betelgeuse in /tmp/git-fat-demo (master)
    $ git fat find 5123123 | cut -d' ' -f3- > /tmp/towel

    darthurdent at betelgeuse in /tmp/git-fat-demo (master)
    $ cat /tmp/towel
    flowerpot.tar.gz
    whale.tar.gz

    darthurdent at betelgeuse in /tmp/git-fat-demo (master)
    $ ll
    total 19M
    drwxrwxr-x 3 darthurdent darthurdent 4.0K Dec 10 13:42 .
    drwxrwxrwt 6 root         root          76K Dec 10 13:42 ..
    drwxrwxr-x 6 darthurdent darthurdent 4.0K Dec 10 13:42 .git
    -rw-r--r-- 1 darthurdent darthurdent 9.0M Dec 10 13:37 flowerpot.tar.gz
    -rw-r--r-- 1 darthurdent darthurdent  10M Dec 10 13:37 whale.tar.gz

Do the ``filter-branch`` using ``git fat index-filter`` as the index filter.
Pass in the file name containing the paths to files you want to exclude.

::

    darthurdent at betelgeuse in /tmp/git-fat-demo (master)
    $ git filter-branch --index-filter 'git fat index-filter /tmp/towel'\
        --tag-name-filter cat -- --all
    Rewrite 28cfba441aac92992c3f80dae97cd1c19b3befad (2/2)
    Ref 'refs/heads/master' was rewritten

Review the changes made to the repository.

::

    darthurdent at betelgeuse in /tmp/git-fat-demo (master)
    $ ll
    total 19M
    drwxrwxr-x 3 darthurdent darthurdent 4.0K Dec 10 13:42 .
    drwxrwxrwt 6 root         root          76K Dec 10 13:42 ..
    drwxrwxr-x 6 darthurdent darthurdent 4.0K Dec 10 13:42 .git
    -rw-rw-r-- 1 darthurdent darthurdent   64 Dec 10 13:42 .gitattributes
    -rw-rw-r-- 1 darthurdent darthurdent 9.0M Dec 10 13:42 flowerpot.tar.gz
    -rw-rw-r-- 1 darthurdent darthurdent  10M Dec 10 13:42 whale.tar.gz

    darthurdent at betelgeuse in /tmp/git-fat-demo (master)
    $ cat .gitattributes
    flowerpot.tar.gz filter=fat -text
    whale.tar.gz filter=fat -text

    darthurdent at betelgeuse in /tmp/git-fat-demo (master)
    $ git cat-file -p $(git hash-object whale.tar.gz)
    #$# git-fat 8c206a1a87599f532ce68675536f0b1546900d7a             10485760

Remove all the old and dangling references by doing a clone of the repository
you just cleaned.  The ``file://`` uri is
`important <http://git-scm.com/book/ch4-1.html>`_ here.

::

    darthurdent at betelgeuse in /tmp/git-fat-demo (master)
    $ cd .. && git clone file://git-fat-demo git-fat-clean

Related projects
----------------

-  `git-annex <http://git-annex.branchable.com>`_ is a far more
   comprehensive solution, but was designed for a more distributed use
   case and has more dependencies.
-  `git-media <https://github.com/schacon/git-media>`_ adopts a similar
   approach to ``git-fat``, but with a different synchronization
   philosophy and with many Ruby dependencies.

Development
-----------

To run the tests, simply run ``python setup.py test``.

To use the development version of ``git-fat`` for manual testing, run
``pip install -U .`` (suggest doing that in a virtualenv).

Master branch is a stable branch with the latest release at the HEAD.


Improvements
------------

-  Better Documentation (esp. setting up a server)
-  Improved Testing
-  cli option to specify which backend to use for push and pull (http, rsync, etc)
-  Python 3 compatibility (without six)
-  Really implement pattern matching
-  Git hooks
