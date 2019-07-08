Git Workflow
============

This guide is for mentors and students in Liftoff. This is a suggestion for how groups in
Liftoff should use Git on their group projects. Groups are also allowed to use their group mentor's
Git workflow if so desired.


What is a Git Workflow?
-----------------------
A git workflow is a strategy for using git. Git is a powerful tool that can be used many ways.
Having a common strategy for using git makes it's easier and more efficient for teams to collaborate.
The are many git workflows that can be adopted by a team. For this course we suggest the
Feature Branch workflow. Details about other git workflows can be found in this
`comparison git workflows <https://www.atlassian.com/git/tutorials/comparing-workflows>`_.


Feature Branch Workflow
-----------------------
The feature branch workflow is focused on keeping work on different features in separate feature branches.
Those features branches are then merged into a master branch.

Master Branch
^^^^^^^^^^^^^
Before discussing how to create feature branches, let's discuss the master branch. The master branch
will be the branch that your feature branches will be created from.

Master branch facts

* The master branch is the main branch for your repository and project.
* The master branch should contain code that is working and ready to be released.

  * Don't merge broken code into master!

* Code should be tested and reviewed before being merged into master.
* Avoid committing directly to the master branch.

.. note::

   Avoid committing directly to master. Instead create a feature branch and merge the branch
   into master using a pull request. Pull requests will be discussed soon.

What is a Feature?
^^^^^^^^^^^^^^^^^^
A **feature** is a requirement or related set of
requirements that add value to a project. Before you can start coding, your team will need to
create a list of features. Sometimes these features are called stories or even
cards if you are using a tool such as Trello. Each feature should be as small as possible, while
still adding value to the project.

Example Features

* Adding a login page
* Fix change password bug
* Make the user list sortable

Feature Branch
^^^^^^^^^^^^^^
When you go to work on a feature, you will need to create a branch specifically for that feature.
Most of the time you will branch off of the master branch.


.. admonition:: Example

   In this example we want to create a feature  branch for the "Fix change password bug" feature.

   1. Make sure you are on the correct source branch before creating the new feature branch.
   2. Pull to make sure that the source branch is up to date.

   ::

      git checkout master
      git pull

   3. Create a new branch with a meaningful name.

      * ``git checkout -b branch-name`` creates and then switches to the new branch

   ::

     git checkout -b fix-change-password-bug

   4. Check your git status to make sure you are on the right branch

   ::

     git status

   **Console Output**

   ::

     On branch fix-change-password-bug
     nothing to commit, working tree clean

Making Commits
^^^^^^^^^^^^^^
You will write code and make commits to implement the feature. Make a commit when you have made
progress towards the feature. Don't commit every time you add a line of code. Don't wait too long
to commit or you will forget what you have done.

Push Your Work to Origin
^^^^^^^^^^^^^^^^^^^^^^^^
After making commits to your feature branch, you will need to push them to origin. This will
make sure they are backed up in case something bad happens to your computer and it will make
the branch visible on origin.


Pull Requests
-------------
A pull request (PR) represents the process of merging (or pulling) a feature branch into master.
A pull request is created and managed on your repository page on github.com. Pull requests are
also referred to as PRs.

.. note::

   Pull requests can be created to merge a branch into any other branch. For the majority
   of this project your pull requests will merge a feature branch into the master branch.


How to Create a Pull Request
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
You can create a PR anytime after you have pushed your feature branch to origin. You can wait
until you are done coding and you have verified that the feature works.

.. admonition:: Example

   This example creates a pull request for the ``add-login-page`` feature branch.

   1. Your feature branch must exist on origin.
   2. Go to the list of branches for your repository.
   3. Click on the New pull request button for the feature branch.

   .. figure:: figures/create-pr-from-branch.png
      :alt: List of branches on Github showing New pull request button.

      New pull request button showing for feature branch.

   4. Do more stuff!!!!!!


Requesting a Review
^^^^^^^^^^^^^^^^^^^
* assign a reviewer


Reviewing a Pull Request
------------------------
* benefits of a code review
* how to review code and leave comments
* when to approve/merge the PR (who merges)
