.. _deploying-java-apps:

Deploying Java Applications
===========================

In this article, we’ll walk through deploying a Spring Boot application
to Pivotal Web Services using Cloud Foundry. Along the way, we’ll
address other topics to consider when deploying an application, such as
database hosting and migration, password management, and network
routing.

.. contents:: Contents
   :local:
   :depth: 2

Requirements
------------

This article assumes you have the following installed:

-  `Java  8 <http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html>`__
-  `Gradle <https://gradle.org/install/>`__
-  `MAMP <https://www.mamp.info/en/>`__
-  :ref:`Cloud Foundry CLI <cloud-foundry-setup>`

Pivotal Web Services Setup
--------------------------

Pivotal Web Services will be the cloud hosting provider for this
article. There are many cloud hosts available to consider (Amazon,
Azure, etc), but we’ll be using Pivotal because it has a lot of support
for deploying Spring Boot applications and is relatively inexpensive.

Create a `Pivotal Account <https://pivotal.io/platform>`__. You will
need to provide a credit card number and a phone number to verify your account.
Later in this guide, we’ll cover how to make sure everything is
un-deployed so you don’t incur any nasty hosting fees.

Once an account has been created, you will want to create a space.
Developers typically use spaces to create segments to their
environments, and may have one for production, development, testing, and
so on. You should have an empty dashboard with a card present to ‘+ Add
a Space’. Do so, and create a space named "Production".

.. _cloud-foundry-setup:

Cloud Foundry Setup
-------------------

We’ll be using a tool called Cloud Foundry to manage our deployment.
Cloud Foundry is widely used in the industry to manage applications and
services. It provides tools to package and deploy our code to production
with minimal configuration.

If you haven’t already, install the `Cloud Foundry
CLI <https://docs.cloudfoundry.org/cf-cli/install-go-cli.html>`__. This
tool will connect our computer to our Pivotal account, and allow us to
deploy and manage our apps.

In a terminal, log into your Pivotal account:

::

   $ cf login

You should see your email, organization name, and the space you just
created on the confirmation message.

Preparing Your Project
----------------------

This article will be based off of the
`cheese-mvc <https://github.com/LaunchCodeEducation/cheese-mvc>`__
project used in the LC101 Java Track.

.. note::

   While we have to choose a specific project for the purposes of illustration, any Spring Boot project could be deployed using the same steps used in this guide.

Clone this project and checkout the ``video-one2many-end`` branch. Base a new branch 
off of this called ``deploy``. If you’d like to keep the
changes we’ll be making, first fork the project to your own repository,
and continue based on the new Git repository url.

Here are the Git commands to carry out these steps.

::

   $ git clone https://github.com/LaunchCodeEducation/cheese-mvc.git
   $ git checkout --track origin/video-one2many-end
   $ git checkout -b deploy

.. tip:: 

   It’s a best practice to create a Git branch solely for the purpose of deploying your application. We’ll use the ``deploy`` branch for this purpose. This will allow for separate configurations to be stored for local development and remote deployment.

   Whenever you want to deploy a new version of your application, you’ll need to merge changes from your main branch (likely ``master``) into ``deploy`` before deploying.


If you don’t have a database user for this project already – you may have
created one when going through unit 3 – you should create one now. Create
a new database user called ``cheese-mvc`` (along with the corresponding
database.) You should now be able to start the application using
``gradle bootRun`` at the command line, from the project root.

Java Artifacts
^^^^^^^^^^^^^^

Typically, when we’re working on our projects, we’re building and running
them locally. We have tools like Gradle and our IntelliJ that help us.
These tools make compiling and running our application simple. We won’t
have those tools in production, however, so we’ll need to package up our
code into an executable JAR file.

A JAR file is a **Java archive**, which is essentially a ZIP file
containing a bunch of compiled Java classes. We could use the ``javac``
command to take our source files and build an executable JAR file.
However, when the JAR is executed, it also needs to be able to find all
the dependencies (for instance, Spring) at runtime in order to use them.
We’ll need to build our JAR file using Gradle so that these dependencies
are packaged up alongside the compiled Java code that we wrote.

Spring Boot adds a plugin to Gradle that allows us to package up
our source code alongside any dependencies, creating a **fat JAR** that
contains compiled versions of our code and all of its dependencies. This
JAR file can then be deployed to our server, and run anywhere that has
the `JVM <https://en.wikipedia.org/wiki/Java_virtual_machine>`__
installed. Try building a fat JAR yourself by running

::

   $ gradle assemble

This created a fat JAR which we can run from the command line with the
command:

::

   $ java -jar build/libs/cheese-mvc-0.0.1-SNAPSHOT.jar

Our application is now ready to be run on a remote machine. Our next
step is to prepare Cloud Foundry to host our application.

Creating your manifest
^^^^^^^^^^^^^^^^^^^^^^

In order to deploy an application with Cloud Foundry, you first must
define what your application is and how it should be run. Cloud Foundry
uses a ``manifest.yml`` file to manage this configuration. Create a file
at the root of the project named ``manifest.yml``. And add the following
lines.

.. code:: yaml

   applications:
   - name: cheese-mvc
     buildpack: java_buildpack
     path: build/libs/cheese-mvc-0.0.1-SNAPSHOT.jar

The ``name`` will specify the unique name for our application, in this
case ``cheese-mvc``. The ``buildpack`` tells Cloud Foundry what type of
application this is and how to manage it. The ``path`` is where to find
our executable project, the fat JAR. If you’re following along with
another project, check ``build/libs/`` to find the name of the JAR you
just built.

Let’s try and deploy our app using:

::

   $ cf push

This will fail. To find out why, let’s try our hand with a little
debugging.

::

   $ cf logs cheese-mvc --recent

Now we can view the recent logs from our app (denoted by the
``--recent`` flag). Does anything here look familiar? You’ll see that
our app can’t connect to the database, which makes sense, since we
haven’t set up a remote database for our app.

Configure the App for a Cloud Database
--------------------------------------

If you had a project without any service dependencies (like a database),
the above steps would be all you need to deploy your application. In our
case, we want to also deploy a database. Luckily, Pivotal provides an
easy (and free) way to setup a MySQL database in our cloud environment.

First, we need to make some changes to our properties to support our
changes. Update your ``application.properties`` so it matches the
following.

::

   # Hibernate ddl auto (create, create-drop, update, none)
   # In this case, we'll want the default behavior to do nothing
   spring.jpa.hibernate.ddl-auto = none

   # Limit the number of active database connections
   # Cloud Foundry's Spark databases can only provide up to four connections
   spring.datasource.tomcat.max-active = 4

The first property is ``spring.jpa.hibernate.ddl-auto``, which we’ll
change to ``none``. Previously, we let Hibernate manage our
database - creating and updating tables as our model classes change. This is
great for testing, as it allows us to add and change our database schema
on the fly. But in the real world, we have to be careful to maintain our
data in production and be very intentional in the changes that we make
to our database. Allowing tools like Hibernate to automatically modify a
database schema is dangerous. To manage our remote database more
manually, we’ll use another tool in a moment to help manage how our
database is configured.

The second property, ``spring.datasource.tomcat.max-active``, is used to
limit our active connections to our database. Typically, Spring Boot
sets reasonable defaults to the number of active connections, but the
database service we will use (Pivotal’s Spark) only allows four
connections at a time.

Flyway database migration
^^^^^^^^^^^^^^^^^^^^^^^^^

Flyway is a tool that uses SQL scripts to create, change, and migrate
database schemas and data. These SQL scripts will live alongside our
project and will provide a way for us to easily recreate the structure
of our database, as well as make additional changes in the future.
Flyway tracks scripts that have been executed and detects new scripts
and runs them at startup.

First, we add the following dependency to our ``build.gradle`` file, in
the ``dependencies`` section.

::

   compile('org.flywaydb:flyway-core')

Spring Boot will detect this dependency and automatically start using
Flyway. We will also need to provide our application with SQL scripts
that specify *how* Flyway should manage our database.

Create a directory in your project called
``src/main/resources/db/migration`` and create a file named in this
directory named ``V1__initialize.sql``. Note that this file name:

-  Starts with a capital V followed by the migration #. Your second migration would start with V2.
-  Has two underscores between the version and the rest of the file name.
-  Has a descriptive *tail* that makes it easy to tell what this migration includes. For example, you might name your second migration something like ``V2__add_user_profile.sql``

Flyway will load these alphabetically and apply our changes. Since
Flyway keeps track of which migrations it has performed previously, it
is important to not change this file once deployed. Instead, to make a
change to your database, create a new file using the naming conventions
above (e.g. ``V2__my_new_change.sql``).

The easiest way to create our initialization script is to export our
existing ``cheese-mvc`` schema. To do this, open PhpMyAdmin, select our
database, and choose the export tab. We then hit *OK* and copy the
generated SQL and add it to our ``V1__initialize.sql`` file. This will
include any test data you have already created, so you may want to clean
up any ``INSERT`` statements if you’d like to start with a clean slate.

An example ``V1__initalize.sql`` might look like:

.. code:: sql

   -- phpMyAdmin SQL Dump
   -- version 4.6.5.2
   -- https://www.phpmyadmin.net/
   --
   -- Host: localhost:8889
   -- Generation Time: Oct 16, 2017 at 10:54 PM
   -- Server version: 5.6.35
   -- PHP Version: 7.0.15

   SET SQL_MODE = "NO_AUTO_VALUE_ON_ZERO";
   SET time_zone = "+00:00";

   --
   -- Database: `cheese-mvc`
   --

   -- --------------------------------------------------------

   --
   -- Table structure for table `category`
   --

   CREATE TABLE `category` (
     `id` int(11) NOT NULL,
     `name` varchar(15) NOT NULL
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

   -- --------------------------------------------------------

   --
   -- Table structure for table `cheese`
   --

   CREATE TABLE `cheese` (
     `id` int(11) NOT NULL,
     `description` varchar(255) NOT NULL,
     `name` varchar(15) NOT NULL,
     `category_id` int(11) DEFAULT NULL
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

   -- --------------------------------------------------------

   --
   -- Indexes for table `category`
   --
   ALTER TABLE `category`
     ADD PRIMARY KEY (`id`);

   --
   -- Indexes for table `cheese`
   --
   ALTER TABLE `cheese`
     ADD PRIMARY KEY (`id`),
     ADD KEY `FK8tkjj8elvx1xu75dpuci756h8` (`category_id`);

   --
   -- AUTO_INCREMENT for table `category`
   --
   ALTER TABLE `category`
     MODIFY `id` int(11) NOT NULL AUTO_INCREMENT;
   --
   -- AUTO_INCREMENT for table `cheese`
   --
   ALTER TABLE `cheese`
     MODIFY `id` int(11) NOT NULL AUTO_INCREMENT;

   --
   -- Constraints for table `cheese`
   --
   ALTER TABLE `cheese`
     ADD CONSTRAINT `FK8tkjj8elvx1xu75dpuci756h8` FOREIGN KEY (`category_id`) REFERENCES `category` (`id`);

Creating a database service
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Next, let’s configure a **service** inside Cloud Foundry for our
database. Cloud Foundry considers anything that isn’t an application to
be a service, and can provide pre-configured services to help speed up
deployment. In this case, we want to create a MySQL database service for
our application. Pivotal has a database called ``cleardb``, a flavor of
MySQL designed for cloud hosting. The *spark* instance size is free and
will be fine for almost all student projects. Creating a service in
Cloud Foundry will also manage passwords and inject them into the
application. This means that we do not need to check in our database
credentials into version control (because we shouldn’t!).

Create a new service:

::

   $ cf create-service cleardb spark cheese-mvc-db

Check running services:

::

   $ cf services

You should see your newly created service displayed but no application
tied to it.

Service binding
^^^^^^^^^^^^^^^

Now you need to attach our application to your service. Cloud Foundry
calls this **service binding** and provides a network connection between
the application and the service. This also removes the need for us to
manually define and configure our database URLs.

::

   $ cf bind-service cheese-mvc cheese-mvc-db

We can confirm this has been configured by running ``cf services``
again, and we should see our app name has been added.

Once this has been established, it does not need to be re-bound when we
redeploy.

Adding a service to the manifest
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Above, we manually configured our service binding for our application.
Alternatively, the ``manifest.yml`` can bind services as well. Update
your ``manifest.yml`` to the following:

::

   applications:
   - name: cheese-mvc
     buildpack: java_buildpack
     path: build/libs/cheese-mvc-0.0.1-SNAPSHOT.jar
     services:
       - cheese-mvc-db

Deploying With Our Database
---------------------------

Now that there is a database service, and matching service binding,
deploy your app again:

::

   $ cf push

We should see a message that our application has started. Open up the
`Pivotal Control panel <https://console.run.pivotal.io/>`__, select your
space, and find your application. Click the *Route* Tab to see the
public URL of your application. In my case, it was
``https://cheese-mvc.cfapps.io``. But yours may be something else, since
these must be unique and are generated for you. Be sure to append
``/cheese`` onto the end of the url (i.e.
``https://cheese-mvc.cfapps.io/cheese``, and try out your newly deployed
app! Cloud Foundry has handled all of the network routing for us, so we
don’t have to configure this ourselves.

Shut It Down
------------

All good things must come to an end. Or, at the very least, will be
billed hourly. When you’re not showing off your new application, it’s
economical to stop the service.

To stop an application:

::

   $ cf stop cheese-mvc

And when you’re ready to use it again

::

   $ cf start cheese-mvc

Since the *spark* instance of our database is free, we can leave it
running. This way we can always have some test data in our application
to demo. However, if you like, you can remove the database service as
well.

To do so, first remove the service binding, which ties the database to
the application:

::

   $ cf unbind-service cheese-mvc cheese-mvc-db

Then, remove the database service:

::

   $ cf delete-service cheese-mvc-db

And then you can delete your application entirely:

::

   $ cf delete cheese-mvc

Scripting
---------

Once you start making changes to your own applications, you are going to
want to be able to easily deploy your application. It may be helpful to
build some helper scripts, so that you don’t have to remember all the
right commands. Here is an example of a deployment script you can use:

``deploy.sh``:

.. code:: bash

   #!/usr/bin/env bash

   # Clean up any old artifacts, and rebuild the jar
   gradle clean assemble

   # Create the database (will give a warning if already exists)
   cf create-service cleardb spark cheese-mvc-db

   # Deploy to cf
   cf push

Mac and Linux users should be able to deploy the application via
``sh ./deploy.sh``. Windows users, if you have Git Bash installed, open
a new Bash prompt and do the same.

Alternatively, you can make the script executable using
``chmod +x deploy.sh``. Then it will be directly executable via
``./deploy.sh``.

You can script the destruction of your application as well. These
commands will completely delete your application, database, and any
associated information, so be careful with them!

``destroy.sh``:

.. code:: bash

   #!/usr/bin/env bash

   # Stops the running app
   cf stop cheese-mvc

   # Removes the binding between services
   cf unbind-service cheese-mvc cheese-mvc-db

   # Removes the db service
   cf delete-service cheese-mvc-db -f

   # Deletes the app entirely
   cf delete cheese-mvc -f

Further Learning
----------------

**How can I keep a set of properties for local development to connect
to my database?** Using spring profiles, and two sets of property files,
you can create properties for both local and deployment. `Read more
about profiles in
Spring <https://docs.spring.io/spring-boot/docs/current/reference/html/howto-properties-and-configuration.html#howto-change-configuration-depending-on-the-environment>`__.

**How can I work with logs for my deployed Java application?** Logging
gives valuable info about your running application, including the
potential sources of errors. :ref:`Read more about logging in
Java <java-logging>`.
