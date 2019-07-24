.. _git-refresher:

Git Refresher
=============

Git is a powerful tool that can be overwhelming to use at times. This guide is a reminder of Git basics
and some best practices.


Read the Git Messages!
----------------------
When using Git in a terminal, Git tries to help you out by giving detailed messages. Read those messages
carefully. If a command isn't working as expected, take your time and see what Git is telling you.
Sometimes those messages will tell you exactly how to fix your issues. If the Git message doesn't solve
your issue, it can help you know what terms to search for when looking for help.


Cloning the Repository
----------------------
When starting on the project, you will first need to clone the remote repository to your computer.
You will clone the repository by going to the repository page on Github and clicking the clone button.
The clone menu will provide two options. For the group project, we suggest using SSH, however cloning
with HTTPS will also work.

Both options will create a Git repository on your local computer with all the files, branches, and commits
that are present in the remote repository.

Cloning Options:

#. SSH

   * Uses SSH to clone the repository.

     * Example: ``git clone git@github.com:LaunchCodeEducation/liftoff.git``

   * Requires a file to be created and uploaded to Github.
   * You do NOT have to enter your password when pushing commits to a remote repository.
   * `Instructions for using SSH <https://help.github.com/en/articles/connecting-to-github-with-ssh>`_

#. HTTPS

   * Uses an HTTPS url for the repository.

     * Example: ``git clone https://github.com/LaunchCodeEducation/liftoff.git``

   * Requires that you enter your password pushing commits to a remote repository.


Keeping Local Repository Up to Date
-----------------------------------
Right after you clone the remote repository, your local copy will be up to date. However, as the project
progresses new branches and commits will be pushed to the remote repository. Your local repository
will NOT automatically update itself.

``git fetch``
^^^^^^^^^^^^^
The fetch command makes your local repository aware of all the new commits and branches on the remote repository.
Git fetch does NOT update any of your local branches, it brings your local Git repository's data about the
available branches and commits up to date.

``git pull``
^^^^^^^^^^^^
The pull command is used to update your current branch with new commits that have been pushed to that branch on
the remote repository. The pull command is a combination of two other Git commands. First, a ``git fetch`` runs
to make sure you local repository is aware of all the new data on the remote repository. Second, a ``git merge``
runs to merge in any new commits for tne current branch.

Remember that Git does NOT keep your local branches up to date. For group projects, you will need to pull in the commits and branches
being created by your group members.


Creating New Branches
---------------------
As detailed in the :ref:`Git workflow section <git-workflow>`, you should create a new branch for each feature you
are working on. This allows features to be developed in isolation, to avoid disrupting other features that are also
being worked on.

Creating a new branch:

#. Use ``git status`` to make sure you are on the base branch you want. The base branch is usually master.

   * WARNING: if you have uncommitted changes, you can:

     #. Stash them.
     #. Commit them to the current branch.
     #. Leave them and commit them on the new branch.

#. Use ``git checkout -b feature-branch-name`` to create and switch to the new branch.
#. You can now make commits and push this branch to the remote repository.


Committing Your Work
--------------------
You need to have *write* access to the remote repository. If working on a group project, this is setup by the group mentor. After you have write
access, you can push up commits to the remote repository. 

#. Run ``git status`` to verify the changes you made show up as not staged or untracked.

   * It's also a good idea to double check you are on the correct branch.

#. Run ``git add .`` to stage all files for commit.

   * It's best to run this command from the root folder of the repository
   * The ``.`` refers to the current directory.
   * You can stage one file at a time if you need to commit only certain files.

     * In that case, replace the ``.`` with a file name or path.

#. Run ``git status`` to verify that the changes have been staged.
#. Run ``git commit -m "short description"`` to make a commit containing all staged changes.
#. Run ``git status`` to verify that there are no uncommitted changes.


Pushing to Remote Repository
----------------------------
When you make a new branch or new commits, you can push them to the remote repository so that
other people can access them.

1. Run ``git status`` to verify your branch and all changes have been committed.

   * Are you noticing a pattern of using ``git status``?

2. Run ``git push``.

   * This defaults to pushing to ``origin master`` which is the remote repository.

3. Review the Git message to see if the push worked.
  
   * Git may need you to to do a ``git pull`` before pushing.


Cheat Sheet for Committing and Pushing
--------------------------------------
Remember these steps when you need to commit and push your code:

1. ``git status`` - verify branch
2. ``git add .`` - stage changes
3. ``git commit -m "short description"`` - commit changes
4. ``git push`` - push commits to remote repository


Stashing Instead of Committing
------------------------------
It may happen that you go to start a new branch but the results from ``git status`` show that you have
uncommitted changes. Even worse, you may not remember what these changes were for. Instead of committing
the changes to the current branch, you can use ``git stash`` to store those changes for future review.

Read this
`summary of stash command <https://education.launchcode.org/intro-to-professional-web-dev/appendices/git/stash.html>`_
for how to use stash.


Git Commands
------------
For a full list of Git commands and more detailed examples, see the resources section below.

.. list-table::
   :widths: auto
   :header-rows: 1

   * - Command
     - Description
   * - ``git help``
     - Information about git commands
   * - ``git COMMAND -h``
     - Information and options for the specific COMMAND
   * - ``git clone A-URL-GOES-HERE``
     - Clones a remote repository into a local folder
   * - ``git branch -a``
     - Shows all local and remote branches
   * - ``git checkout BRANCH-NAME``
     - Switches the branch you are on to BRANCH-NAME
   * - ``git checkout -b NEW-BRANCH-NAME``
     - Creates a new branch using given name and then switches to that branch
   * - ``git status``
     - Reports the current state of the repo. Make sure to run this at the top level of your repository
   * - ``git add .``
     - Stages (adds) files to be committed. Changes have to be staged before they can be committed
   * - ``git commit -m "short description of changes"``
     - Creates a commit that contains all the staged changes.
   * - ``git push``
     - Pushes local commits to the remote repository. Then the commits can be pulled down by other team members
   * - ``git fetch``
     - Makes your local repository aware of new branches and commits on the remote repository. Does NOT update any local branches, only makes your local aware of new commits.
   * - ``git merge BRANCH-NAME``
     - Merges the commits from BRANCH-NAME into your current branch
   * - ``git pull``
     - Runs a ``git fetch`` and ``git merge``
   * - ``git stash``
     - A way of keeping changes without committing them to your branch.
   * - ``git log``
     - See a list of commits with info about the author, date, and time of each commit
   * - ``git diff OPTION``
     - Will show differences in files between two branches or commits.


Additional Resources
--------------------
* `Official Git Reference <https://git-scm.com/docs>`_ - A Git command reference from the creators of Git.
* `Pro Git Book <https://git-scm.com/book/en/v2>`_ - A reference book covering Git in depth.
* `Flight Rules for Git <https://github.com/k88hudson/git-flight-rules>`_ - A "How to" guide for Git.
* `Interactive GitHub Sandbox <https://try.github.io>`_ - A place to practice Git without fear of messing anything up.
* `Connecting to GitHub with SSH <https://help.github.com/articles/connecting-to-github-with-ssh/>`_
