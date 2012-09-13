Context aware diffs with git
============================

:date: 2009-09-29

Earlier this week Luke Cox asked in response to a patch I sent:

    What version of git_ are you using?  Mine doesn't seem to produce the right
    location output for ``format-patch``.

``git``, by default, displays a `function name in the hunk header`_ of its
``diff`` output.  It produces some really nice output for certain languages, but
out of the box it doesn't display nice information for all the file formats you
may use.

The patch I sent Luke included a significant change to an
`.ini`_ format file, including some mostly
accurate location information in the hunk header.  It wasn't because I use
a newer version of ``git``, just that I've set it up to use different matchers for
different files.  In my ``~/.gitconfig`` I have the following snippet to use
better function names in ``ini`` and ``adr`` files:

.. code-block:: ini

    [diff "ini"]
            funcname = "^\\[.*\\]$"
    [diff "adr"]
            funcname = "^#.*$"

`Fork this code <http://gist.github.com/198037>`__

And, to enable them you must tell ``git`` which files to use the new matchers with
by editing the ``.gitattributes`` file:

.. code-block:: text

    *.ini diff=ini
    *.adr diff=adr

`Fork this code <http://gist.github.com/198038>`__

The ``funcname`` values are simple regular expressions to search for, so in the
``ini`` example it is searching for a line that begins with a ``[`` and ends with
a ``]`` as these are the common section headers.  And the ``adr`` matcher just
specifies a line that begins with a ``#``.  It is important to match the entire
string, as it is the matched content that is used in the diff hunk's output.

.. _git: http://www.git-scm.com/
.. _function name in the hunk header: http://www.gnu.org/software/diffutils/manual/html_node/C-Function-Headings.html
.. _.ini: http://www.cloanto.com/specs/ini.html