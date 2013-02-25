Making a nice home
------------------

:date: 2009-10-11

Back in :doc:`Kick_me_birthday_reminders` I said:

    You could trigger a rebuild in your ``~/.bashrc`` before you call ``rem`` to
    see the reminders, so they are always up to date at login. Or, you could be
    like me and have a post commit hook for git_ to manage this… because you are
    keeping your home directory version controlled as a sanity measure I hope!

Today, Stuart Grady asked me in a private mail:

    Okay, at which point is the "Tip of the Day" thingy going to be about using
    hooks and make_ in ``$HOME``?

I'll take the hint and look at ``make``.  For the purposes of this post we're
going to pretend we haven't read `recursive make considered harmful`_ as I don't
heed its advice for the ``Makefile`` I have in my home directory.  That said,
let's have a look at what can be found in my current top-level ``Makefile``.

Permissions
-----------

I keep my home directory version controlled with ``git``, but git out of the box
doesn't maintain permissions on files beyond the executable bit.  There are
plenty of ways around this including using external tools such as etckeeper_,
but I prefer the simple approach of setting the permissions when they're needed:

.. code-block:: make

    PRIVATE_FILES := .abook/addressbook .gnupg/secring.gpg .mailfilter .msmtprc \
            .ssh/id_rsa
    PRIVATE_DIRS := .gnupg

    fix-perms: $(PRIVATE_FILES) $(PRIVATE_DIRS)
            $(info - Removing extra read permissions from private files and directories)
            chmod 600 $(PRIVATE_FILES)
            chmod 700 $(PRIVATE_DIRS)

`Fork this code <http://gist.github.com/208207>`__

If this rule is called after a ``git pull`` is issued then the files always have
the correct permissions.

vim hacks
---------

I also call ``make`` in some subdirectories, the most interesting one is probably
for ``.vim``:

.. code-block:: make

    CTAGS := exuberant-ctags

    TARGETS := $(patsubst /usr/lib/%, tags/%.ctags, $(wildcard /usr/lib/python*)) \
            doc/tags

    all: $(TARGETS)

    tags/%.ctags: /usr/lib/%
            $(CTAGS) --exclude=test_* --exclude=tests.py --exclude=test.py \
                    --exclude=*/test/* --exclude=*/tests/* --languages=python \
                    -R -f $@ $<

    doc/tags: $(filter-out doc/tags, $(wildcard doc/*))
            vim -X -u NONE -c 'helptags $(dir $@)' -c ':q' </dev/null &>/dev/null

    clean:
            rm -f $(TARGETS)

`Fork this code <http://gist.github.com/208209>`__

The values in the ``patsubst`` block of the ``$(TARGETS)`` definition are a list
of filenames to use for tag storage with ctags_.  The rule creates a different
``ctags`` file for each installed python_ version.  In the rule to make the
``ctags`` files we specifically exclude test files as they aren't often useful
in omni-completion_, and they significantly pollute the tags database for normal
use.

The final entry in ``$(TARGETS)`` simply updates the tag lists for `help files`_
installed in ``~/.vim/doc``.

Documentation
-------------

The final subdirectory ``Makefile`` we're going to look at is actually called in
a number of different directories to create HTML versions of
reStructuredText_ files.

.. code-block:: make

    GENERATED = $(patsubst %.rst, %.html, $(wildcard *.rst))

    all: $(GENERATED)

    $(GENERATED): %.html: %.rst
            rst2html.py $< $@

`Fork this code <http://gist.github.com/208210>`__

If we call the above ``Makefile`` from our ``git`` hooks after any pull or merge we
always have up to date processed versions of documents.

.. _git: http://www.git-scm.com/
.. _make: http://www.gnu.org/software/make/make.html
.. _recursive make considered harmful: http://miller.emu.id.au/pmiller/books/rmch/
.. _etckeeper: http://joey.kitenet.net/code/etckeeper/
.. _ctags: http://ctags.sourceforge.net
.. _python: http://www.python.org
.. _omni-completion: http://vimdoc.sourceforge.net/htmldoc/version7.html#new-omni-completion
.. _help files: http://vimdoc.sourceforge.net/htmldoc/various.html#:helptags
.. _reStructuredText: http://docutils.sourceforge.net/rst.html