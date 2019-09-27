.. _tutorial:

========
Tutorial
========

Features
========

*behave* operates on directories containing:

1. `feature files`_ written by your Business Analyst / Sponsor / whoever
   with your behaviour scenarios in it, and
2. a "steps" directory with `Python step implementations`_ for the
   scenarios.  QE will not write steps to start.


Feature Files
=============

A feature file has a :ref:`natural language format <chapter.gherkin>`
describing a feature or part of a feature with representative examples of
expected outcomes.
They're plain-text (encoded in UTF-8) and look something like:

.. code-block:: gherkin

  Feature: Fight or flight
    In order to increase the ninja survival rate,
    As a ninja commander
    I want my ninjas to decide whether to take on an
    opponent based on their skill levels

    Scenario: Weaker opponent
      Given the ninja has a third level black-belt
       When attacked by a samurai
       Then the ninja should engage the opponent

    Scenario: Stronger opponent
      Given the ninja has a third level black-belt
       When attacked by Chuck Norris
       Then the ninja should run for his life

The "Given", "When" and "Then" parts of this prose form the actual steps
that will be taken by *behave* in testing your system. These map to `Python
step implementations`_. As a general guide:

**Given** we *put the system in a known state* before the
user (or external system) starts interacting with the system (in the When
steps). Avoid talking about user interaction in givens.

**When** we *take key actions* the user (or external system) performs. This
is the interaction with your system which should (or perhaps should not)
cause some state to change.

**Then** we *observe outcomes*.

You may also include "And" or "But" as a step - these are renamed by *behave*
to take the name of their preceding step, so:

.. code-block:: gherkin

    Scenario: Stronger opponent
      Given the ninja has a third level black-belt
       When attacked by Chuck Norris
       Then the ninja should run for his life
        And fall off a cliff

In this case *behave* will look for a step definition for
``"Then fall off a cliff"``.


Scenario Outlines
-----------------

Sometimes a scenario should be run with a number of variables giving a set
of known states, actions to take and expected outcomes, all using the same
basic actions. You may use a Scenario Outline to achieve this:

.. code-block:: gherkin

  Scenario Outline: Blenders
     Given I put <thing> in a blender,
      when I switch the blender on
      then it should transform into <other thing>

   Examples: Amphibians
     | thing         | other thing |
     | Red Tree Frog | mush        |

   Examples: Consumer Electronics
     | thing         | other thing |
     | iPhone        | toxic waste |
     | Galaxy Nexus  | toxic waste |

*behave* will run the scenario once for each (non-heading) line appearing
in the example data tables.


Step Data
---------

Sometimes it's useful to associate a table of data with your step.

Any text block following a step wrapped in ``"""`` lines will be associated
with the step. For example:

.. code-block:: gherkin

   Scenario: some scenario
     Given a sample text loaded into the frobulator
        """
        Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do
        eiusmod tempor incididunt ut labore et dolore magna aliqua.
        """
    When we activate the frobulator
    Then we will find it similar to English

The text is available to the Python step code as the ".text" attribute
in the :class:`~behave.runner.Context` variable passed into each step
function.

You may also associate a table of data with a step by simply entering it,
indented, following the step. This can be useful for loading specific
required data into a model.

.. code-block:: gherkin

   Scenario: some scenario
     Given a set of specific users
        | name      | department  |
        | Barry     | Beer Cans   |
        | Pudey     | Silly Walks |
        | Two-Lumps | Silly Walks |

    When we count the number of people in each department
    Then we will find two people in "Silly Walks"
     But we will find one person in "Beer Cans"


.. _`controlling things with tags`:

Controlling Things With Tags
============================

You may also "tag" parts of your feature file. At the simplest level this
allows *behave* to selectively check parts of your feature set.

Given a feature file with:

.. code-block:: gherkin

  Feature: Fight or flight
    In order to increase the ninja survival rate,
    As a ninja commander
    I want my ninjas to decide whether to take on an
    opponent based on their skill levels

    @slow
    Scenario: Weaker opponent
      Given the ninja has a third level black-belt
      When attacked by a samurai
      Then the ninja should engage the opponent

    Scenario: Stronger opponent
      Given the ninja has a third level black-belt
      When attacked by Chuck Norris
      Then the ninja should run for his life

then running ``behave --tags=slow`` will run just the scenarios tagged
``@slow``. If you wish to check everything *except* the slow ones then you
may run ``behave --tags="not @slow"``.

Another common use-case is to tag a scenario you're working on with
``@wip`` and then ``behave --tags=wip`` to just test that one case.

Tag selection on the command-line may be combined:

* ``--tags="@wip or @slow"``
   This will select all the cases tagged *either* "wip" or "slow".

* ``--tags="@wip and @slow"``
   This will select all the cases tagged *both* "wip" and "slow".

Re-visiting the example from above; if only some of the features required a
browser and web server then you could tag them ``@fixture.browser``:


Works In Progress
=================

*behave* supports the concept of a highly-unstable "work in progress"
scenario that you're actively developing. This scenario may produce strange
logging, or odd output to stdout or just plain interact in unexpected ways
with *behave*'s scenario runner.  NCR uses wip tagged scenarios to mean they are not implemented in automation yet.


