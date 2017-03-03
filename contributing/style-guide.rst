.. _style_guide:

===================
Writing Style Guide
===================

When you write for SmartThings platform, your audience should find your documentation readable, interesting and informative.
To accomplish these goals, we encourage you to stick to our recommended writing style. 

----

Titles and headings
-------------------

Wherever possible, write purpose-driven documentation. 
This means writing document titles and section headings that state the benefit explicitly. 
Such titles or headings can be written as either calls to actions or tasks that can be done. 
This approach makes it easier for the reader to learn how to get her job done. 

**Examples:** 

- Preferred: Writing Your First SmartApp (document title)
- Avoid: SmartApp Fundamentals (document title isn't purpose-driven)
- Preferred: Create your own RESTful API (section heading)
- Avoid: Parent-child SmartApps (document title isn't purpose-driven)
- Preferred: Combine Multiple Automations (document title)

.. note::

    - A **document title** is the main title of a document page. A document has only one document title. Example: "Writing Style Guide" at the beginning of this page. The document title also appears at the top level in the navigation bar, so it must be short, preferably four to five words or less. 
    - A **section heading** is the title for an individual section within a document page. Example: "Titles and headings" at the top of this section. A document page can have multiple sections, and hence multiple section headings. 

Avoid framing as questions
^^^^^^^^^^^^^^^^^^^^^^^^^^

Avoid using questions in document titles and section headings. 

**Example:** 

- Avoid: How does the switch turn on?
- Preferred: How the switch turns on (section heading)

Avoid italics (emphasis)
^^^^^^^^^^^^^^^^^^^^^^^^

Avoid using emphasis (italics) in document titles or section headings.
See :ref:`page_structure`.

----

Document titles
^^^^^^^^^^^^^^^

Use title case, as in “Document Titles” and not “Document titles.” 

**Example:**

- The title of this document, "Writing Style Guide."

What not to capitalize in title
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- as (SmartApp as a Wakeup Device)
- to (How to Subscribe to Events)
- on (Trigger on Time)
- at (Alarm at a Specific Time)
- of, the (No More Rules of the Game)
- in (Your First SmartApp in Five Minutes)
- the (Motion at the Garage Door)
- a (Three Critical Triggers in a Day)
- for (Temperature Control for Basement)

What to capitalize in title
^^^^^^^^^^^^^^^^^^^^^^^^^^^

- is (Device Health Is Reported Weekly)
- was (Motion Was Detected)
- then (Set a Sunrise Automation, Then Sit Back)
- up (Biff Stood Up to Trigger the Alarm)
- that (The Battery That Lasts Forever)
- with (The Day Ends With Cheers All Around)

Section headings
^^^^^^^^^^^^^^^^

Use sentence case, as in “Subscription management,” and not the title case, as in “Subscription Management.” 

.. note::

    - A **sentence case** is where you only capitalize the first letter of the sentence. 
    - A **title case** is where you capitalize first letter of every word of the sentence. 

----

.. _ui_elements:

UI elements
-----------

Always use italics emphasis when quoting a UI element label such as a button label or an icon label.

**Example:** 

    Go to the *Simulator* menu, and click on *Browse SmartApp Templates* in the dropdown list. 

Here it is in reStructuredText:

    Go to the \*Simulator* menu, and click on \*Browse SmartApp Templates* in the dropdown list.

.. _page_structure:

Page structure
--------------

Each document should be named with a ``.rst`` file extension.
Each page is composed of a title, followed by some short text outlining the purpose of the document.

Sections should be delimited by ``----``, to insert a line separator.

The structure should look like this:

.. code-block:: rst

    ==========
    Page Title
    ==========

    Some introductory material.

    ----

    Section 1
    ---------

    Section text.

    ----

    Section 2
    ---------

    Section text.

    Subsection 2.1
    ^^^^^^^^^^^^^^

    Subsection text.



Page title
^^^^^^^^^^

Page titles appear at the top of the document, and have a row of ``===`` characters above and below.
Page titles should have title capitalization:

.. code-block:: rst

    ====================
    This is a Page Title
    ====================

Headings
^^^^^^^^

Top-level section headings are followed by a row of ``---`` characters.
They should have sentence capitalization:

.. code-block:: rst

    This is a section
    -----------------

Subsection headings are followed by a row of ``^^`` characters.
They should have sentence capitalization.

.. code-block:: rst

    This is a section
    -----------------

    This is a subsection
    ^^^^^^^^^^^^^^^^^^^^

.. note::

    Not all documents currently follow the guideline of using ``^^^`` for subsections.
    If you are editing a document and see a different heading syntax, feel free to change it.

----

reStructuredText syntax
-----------------------

Links
^^^^^

Links to external targets look like this:

.. code-block:: rst

    `SmartThings <http://smartthings.com>`_

Links to sections within the document can be included like this:

.. code-block:: rst

    Section name
    ------------

    See `Other section`_ for more information.

    Other section
    -------------

The ``:ref:`` target allows us to link to other documents or document sections.
It requires placing a label above a section, title, or image:

.. code-block:: rst

    .. _section_label:

    Some section
    ------------

Another document can then link to ``Some section`` like this:

.. code-block:: rst

    See :ref:`section_label` for more information.

Lists
^^^^^

Ordered lists appear like this:

.. code-block:: rst

    #. Item 1
    #. Item 2
    #. Item 3

Which results in:

#. Item 1
#. Item 2
#. Item 3

Unordered lists use a ``-`` or ``*`` character:

.. code-block:: rst

    - First bullet
    - Second bullet

Inline markup
^^^^^^^^^^^^^

- Surround text with \* for *italics text*.
- Surround text with \** for **strong text**.
- Surround text with \`` for code samples (``someMethod()``).

When referring to method calls in the documentation, place ``()`` after the method name: ``methodName()``.
This helps distinguish methods from other code literals.

Code examples
^^^^^^^^^^^^^

Code blocks can be included using the ``code-block`` directive.
Use the appropriate language for the code sample.
Code blocks may appear with line numbers (use ``:linenos:``) and may emphasize certain lines:

.. code-block:: rst

    .. code-block:: groovy
        :linenos:
        :emphasize-lines: 3

        def someMethod() {
            def myVar = 14
            doSomethingAmazing(myVar)
        }

The above code block renders as:

.. code-block:: groovy
    :linenos:
    :emphasize-lines: 3

    def someMethod() {
        def myVar = 14
        doSomethingAmazing(myVar)
    }

Images
^^^^^^

Images are found in the ``/img`` directory of the documentation, and can be included like this (you may need to alter the path depending on the location of the document):

.. code-block:: rst

    .. image:: ../img/getting-started/building-img.png

The above will render as:

.. image:: ../img/getting-started/building-img.png

Admonitions
^^^^^^^^^^^

Admonitions are ways of calling out certain bodies of text:

.. code-block:: rst

    .. note::

        A note provides more information about the content, in a side-bar like format.

    .. tip::

        A tip is some extra information that while not strictly necessary, may lead to the reader learning a new way of doing something.

    .. warning::

        A warning is just that - a warning of something that the reader should be aware of.

    .. error::

        An error is for error conditions.

The above results in:

.. note::

    A note provides more information about the content, in a side-bar like format.

.. tip::

    A tip is some extra information that while not strictly necessary, may lead to the reader learning a new way of doing something.

.. warning::

    A warning is just that - a warning of something that the reader should be aware of.

.. error::

    An error is for error conditions.


Tables
^^^^^^

Simple tables in RST look like this:

.. code-block:: rst

    ========= =========
    Heading 1 Heading 2
    ========= =========
    1.1       1.2
    2.1       2.2
    ========= =========

The above renders as:

========= =========
Heading 1 Heading 2
========= =========
1.1       1.2
2.1       2.2
========= =========

Grid tables can be written like this:

.. code-block:: rst

    +------------+------------+-----------+
    | Header 1   | Header 2   | Header 3  |
    +============+============+===========+
    | body row 1 | column 2   | column 3  |
    +------------+------------+-----------+
    | body row 2 | Cells may span columns.|
    +------------+------------+-----------+
    | body row 3 | Cells may  | - Cells   |
    +------------+ span rows. | - contain |
    | body row 4 |            | - blocks. |
    +------------+------------+-----------+

Which results in:

+------------+------------+-----------+
| Header 1   | Header 2   | Header 3  |
+============+============+===========+
| body row 1 | column 2   | column 3  |
+------------+------------+-----------+
| body row 2 | Cells may span columns.|
+------------+------------+-----------+
| body row 3 | Cells may  | - Cells   |
+------------+ span rows. | - contain |
| body row 4 |            | - blocks. |
+------------+------------+-----------+

----

Miscellaneous tips
------------------

- Spell check before committing.
- Show, don't tell - include example code.
- Place each sentence on a new line to help with review and readability.
- Not all documents currently follow these guidelines. See the `Contributing <https://github.com/SmartThingsCommunity/Documentation/blob/master/README.md>`_ guide to learn how you can contribute, and help address that. :)

----

Further reading
---------------

- `Sphinx documentation <http://sphinx-doc.org/contents.html>`_
- `reStructuredText Reference <http://docutils.sourceforge.net/docs/user/rst/quickref.html>`_
