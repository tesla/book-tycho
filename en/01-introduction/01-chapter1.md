Introduction
============

Tycho is a set of Maven extensions (aka plugins) dedicated to build OSGi
bundles, p2 repositories, as well as other Eclipse constructs such as
features and RCP applications. Because Tycho is part of the Maven
ecosystem, users can leverage the rich Maven ecosystem of plugins for
code generators, quality analysis tools, and code coverage.

How old is tycho?
=================

Tycho started a couple of years ago out the need to have the ability to
build Eclipse plugins using Maven. Tom Huybrechts started the initial
work on Tycho. Sonatype joined the forces shortly after that. Currently,
the main contributors to Tycho are SAP, Sonatype, and Intallio.

Where is tycho hosted?
======================

Tycho is an open source project run by Sonatype. It has recently been
given to the Eclipse Foundation, where it will be moved in a near
future. It is being promoted as the build technology of choice to help
with the long-term support strategy that the Eclipse Foundation is
putting together.

Currently the Tycho codebase can be found on github:
[github.com/sonatype/sonatype-tycho](https://github.com/sonatype/sonatype-tycho),
the users and development mailing lists are available at:
[tycho-users@lists.sonatype.com](mailto:tycho-users@lists.sonatype.com)
and bugs can be filed at
[issues.sonatype.org/browse/TYCHO](https://issues.sonatype.org/browse/TYCHO).

Tycho and other technologies
============================

This section positions Tycho w.r.t other technologies in the context of
which it is often mentioned.

Eclipse PDE UI / PDE Build PDE is the technology provided by the
----------------------------------------------------------------

Eclipse Platform team to build bundles, and other Eclipse constructs.

It is composed of two parts:

-   A "UI" part, referred to as PDE UI, that provides the classpath
    management infrastructure in the IDE, the launching capabilities
    (e.g. Run as OSGi, Run as Eclipse Application), as well as editors
    for OSGi manifest, features, product files, target platforms, etc.

-   A "headless" part, referred to as PDE Build, which since the
    inception of Eclipse, provides Ant scripts and Ant script generators
    to build plugins, features and RCP applications outside of the IDE

-   Tycho, being a headless build mechanism "competes" with PDE Build in
    the sense that it provides an alternate way to build plugins and
    other Eclipse artifacts. That being said, Tycho reuses some of the
    files that are being used by PDE Build, such as build.properties,
    manifest.mf, product files, which allows one to reuse PDE UI
    infrastructure.

Maven bundle plugin and Bnd
---------------------------

The Maven bundle plugin is a Maven plugin
([http://felix.apache.org/site/apache-felix-maven-bundle-plugin-bnd.html](http://felix.apache.org/site/apache-felix-maven-bundle-plugin-bnd.html))
that uses Bnd
([http://www.aqute.biz/Code/Bnd](http://www.aqute.biz/Code/Bnd)) to
generate an OSGi manifest from an analysis of the jar classfiles. This
approach allows the user to specify its dependencies in the pom.xml
rather than the manifest. Because it uses the Maven way of specifying
dependencies in the pom.xml, this approach is often referred to as
“pom-first mode”, to contrast with the “manifest-first mode” that is put
forward by Tycho and PDE.

At this point the Maven bundle plugin only deals with the generation of
manifests and does not produce p2 repositories or understand other
Eclipse constructs.

p2
--

p2 is the codename of Eclipse and OSGi update mechanism. It is not a
build mechanism. Tycho relates to p2 in three ways.

-   Tycho produces p2 repositories as part of a build.

-   Tycho downloads the dependencies of the entities being built from p2
    repositories. By default, Tycho does not resolve its dependencies
    from the Maven repository because the Maven dependency and
    repository models can not accommodate the expression of OSGi
    dependencies like import packages.

-   Tycho embeds parts of p2 in order to perform the dependency
    resolution and a few other key operations.

Tycho (Maven) / Hudson / Nexus
------------------------------

In the bigger picture, Tycho (Maven) is a build engine in the sense that
it is the entity that actually compiles code and runs tests. It can run
anywhere, on your local machine or on your Continuous Integration server
(CI) such as Hudson. Hudson plays the role of a "scheduler" in that it
triggers the execution of the Tycho build, based on some triggers
(manual, scheduled, SCM change).

Typically, when run on a CI server, the artifacts being built (for
example, jars, zip, sources) are made available on a repository manager
such as Nexus. Beyond just storing those artifacts, a repository manager
makes available the artifacts for other builds to consume. So to recap,
a Tycho build runs on a Hudson server, and obtains and publishes
artifacts from a Nexus server. TO FILL DRAWING

Who is using Tycho?
===================

Many companies including SAP, Sonatype, ZeroTurnaround, Intallio, … as
well as open source projects of all sizes and complexity use Tycho in
production.

Building an OSGi bundle / Eclipse plug-in
=========================================

Building an OSGi bundle / Eclipse plug-in
=========================================

In this chapter we are reviewing the simplest - though not most concise
- way of building an OSGi bundle. The example shown in this section is
not meant to be used as a starting point, but used solely in the context
of an introduction to group together all the relevant bits of
configuration in one place.

As shown in the following picture (TO FILL ref to files.png), beside the
source code, the example contains three relevant files:

-   The OSGi Manifest.mf, which captures the dependencies of the bundle
    being built.

-   The build.properties, which describes the set of files that will be
    included in the final archive (bin.includes property)

-   The pom.xml, which indicates to Maven how to build this project.

Given that Manifest.mf and build.properties are known from PDE users,
this section will mostly explain the pom.xml presented below.

    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                           http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
      <groupId>tycho.tutorial</groupId>
      <artifactId>example1</artifactId>
      <version>1.0.0-SNAPSHOT</version>
      <packaging>eclipse-plugin</packaging>

      <properties>
        <tycho-version>0.11.0</tycho-version>
      </properties>

      <repositories>
        <!-- configure p2 repository to resolve against -->
        <repository>
          <id>helios</id>
          <layout>p2</layout>
          <url>http://download.eclipse.org/releases/helios</url>
        </repository>
      </repositories>

      <build>
        <plugins>
          <plugin>
            <!-- enable tycho build extension -->
            <groupId>org.sonatype.tycho</groupId>
            <artifactId>tycho-maven-plugin</artifactId>
            <version>${tycho-version}</version>
            <extensions>true</extensions>
          </plugin>
          <plugin>
            <groupId>org.sonatype.tycho</groupId>
            <artifactId>target-platform-configuration</artifactId>
            <version>${tycho-version}</version>
            <configuration>
              <!-- recommended: use p2-based target platform resolver -->
              <resolver>p2</resolver>
            </configuration>
          </plugin>
        </plugins>
      </build>
    </project>

Identifying what is being built
===============================

    <groupId>tycho.tutorial</groupId>
    <artifactId>example1</artifactId>
    <version>1.0.0-SNAPSHOT</version>

groupId
:   This identifier is a necessity to "make the Maven gods happy". In a
    non-tycho Maven build, the groupId is used to organize the artifacts
    in a namespace. However, in Tycho its importance is reduced since
    usually the Bundle-SymbolicName is a fully qualified name (e.g.
    org.eclipse.platform) that is not ambiguous.

artifactId
:   artifactId is the name given to what is being built. It needs to be
    a copy of the Bundle-SymbolicName attribute, as found in
    manifest.mf.

version
:   version is the version of the artifact being built. It must match
    the Bundle-Version attribute, as found in the manifest.mf. A slight
    twist here comes from the fact that versions used in manifest end
    with .qualifier and that the one specified in the pom.xml end with
    -SNAPSHOT. For example 1.0.0.qualifier becomes 1.0.0-SNAPSHOT.

The tuple, groupId, artifactId, version is also referred to as GAV or
coordinate.

The duplication of information between the Manifest.mf and the pom.xml
is unfortunate however it is one that we have to live with for the time
being. This repetition can be the cause of build failures when the
values are not in sync, and it would lead to the following message:

    [ERROR] Failed to execute goal
    org.sonatype.tycho:maven-osgi-packaging-plugin:0.11.0:validate-version
    (default-validate-version) on project example1: Unqualified OSGi
    version 1.0.0.qualifier must match unqualified Maven version
    0.0.1-SNAPSHOT for SNAPSHOT builds -> [Help 1]

The kind of entity being built
==============================

This part of the markup tells Maven that what is being built is an
Eclipse plugin. This packaging type is not specific to Eclipse and
should be used to build OSGi bundles. Tycho defines additional packaging
types that will be presented in the following chapters.

    <packaging>eclipse-plugin</packaging>

Repositories
============

In order to satisfy the dependencies expressed in the Manifest.mf and
thus successfully build the bundle, Tycho needs to access p2
repositories. The identification of these repositories is done using the
repository markup as defined by Maven. For example, the following markup
will cause the Eclipse Helios repository to be used to revolve
dependencies. Note that it is important to set the layout to be p2,
since it is what indicates to Maven that this is not a regular maven
repository.

    <repositories>
      <!-- configure p2 repository to resolve against -->
      <repository>
        <id>helios</id>
        <layout>p2</layout>
        <url>http://download.eclipse.org/releases/helios</url>
      </repository>
    </repositories>

The build section
=================

The build section
([http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html](http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html))
of the pom.xml instructs Maven how to configure the execution of various
build phases. In this section we are reviewing the boilerplate markup
that is necessary to configure Tycho.

First, the reference to the tycho-maven-plugin, indicates that Tycho is
a Maven extension that hooks in Maven at a very low level.

    <build>
      ...
      <plugins>
        ...
        <plugin>
          <groupId>org.sonatype.tycho</groupId>
          <artifactId>tycho-maven-plugin</artifactId>
          <version>0.11.0-SNAPSHOT</version>
          <extensions>true</extensions>
        </plugin>

Second is the indication that the p2 resolver should be used.

    <build>
      ...
      <plugins>
        ...
        <plugin>
          <groupId>org.sonatype.tycho</groupId>
          <artifactId>target-platform-configuration</artifactId>
          <version>0.11.0-SNAPSHOT</version>
          <configuration>
            <!-- recommended: use p2-based target platform resolver -->
            <resolver>p2</resolver>
          </configuration>
        </plugin>

Controlling the content of the final archive
============================================

The output of this build is a jar. In order to control what is added to
the final add, Tycho will use the value of the bin.includes property as
defined in the build.properties.

Executing the build
===================

The execution of the build is trivial, since it only requires an
installation of Maven 3 and to type in "mvn clean install" in the folder
that contains the pom.xml.

When the build is running, a lot of information will be displayed in the
console. From a high level, it will first read the manifest.mf, connect
to repositories, resolve dependencies, download and cache necessary
bundles from p2 repository, compile and finally create the final jar. A
successful build will end with the message "BUILD SUCCESS" and a failed
build with "BUILD FAILURE".

The result of the build is stored in the target folder at the root of
the plugin being built (see screenshot afterBuild.png). In the case of
our example a file called `example1-1.0.0-SNAPSHOT.jar` can be found.
The target folder contains other files that have been created as part of
the build.

Toward a recommended project structure
======================================

In this section we are evolving the simplistic example provided in the
previous section toward a recommended structure for Tycho projects.

Introducing the parent pom
==========================

Looking at the XML from the previous section, it appears obvious that if
we had a second plugin to build, the solution would not scale very well
since a lot of XML would have to be duplicated. This duplication problem
gets solved using the concept of parent POM
([http://sonatype.com/books/maven-book/reference/pom-relationships-sect-project-inheritance.html](http://sonatype.com/books/maven-book/reference/pom-relationships-sect-project-inheritance.html))
that exists in Maven. Through inheritance Maven projects can inherit
values defined in parents, thus allowing several projects to share the
same configuration and alleviating the need for duplication.

The following xml snippet is the complete parent that is derived from
the previous example. As you can observer, the build section and the
repository sections are now moved there since they are common to the
projects being built.

    <?xml version="1.0" encoding="UTF-8"?>
    <project
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          http://maven.apache.org/xsd/maven-4.0.0.xsd"
      xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
      <modelVersion>4.0.0</modelVersion>
      <groupId>tychodemo</groupId>
      <artifactId>parent</artifactId>
      <version>0.0.1-SNAPSHOT</version>
      <packaging>pom</packaging>

      <properties>
        <tycho-version>0.11.0-SNAPSHOT</tycho-version>
      </properties>
      <repositories>
        <!-- configure p2 repository to resolve against -->
        <repository>
          <id>helios</id>
          <layout>p2</layout>
          <url>http://download.eclipse.org/releases/helios</url>
                <!-- file URL for faster and offline builds -->
          <!-- <url>file:/${basedir}/../../helios</url> -->
        </repository>
      </repositories>
      <build>
        <plugins>
          <plugin>
            <!-- enable tycho build extension -->
            <groupId>org.sonatype.tycho</groupId>
            <artifactId>tycho-maven-plugin</artifactId>
            <version>${tycho-version}</version>
            <extensions>true</extensions>
          </plugin>
          <plugin>
            <groupId>org.sonatype.tycho</groupId>
            <artifactId>target-platform-configuration</artifactId>
            <version>${tycho-version}</version>
            <configuration>
              <!-- recommended: use p2-based target platform resolver -->
              <resolver>p2</resolver>
            </configuration>
          </plugin>
        </plugins>
      </build>
    </project>

The packaging type of a parent is pom, and each child will specify its
own packaging type.

The version of Tycho is factored out in a property called tycho-version.
It is usually a good practice because it makes it easy to consume a new
version of Tycho without having to update several places Note that this
practice of using properties is not only limited to Tycho and is widely
used in Maven
([http://maven.apache.org/guides/introduction/introduction-to-the-pom.html\#Project\_Interpolation](http://maven.apache.org/guides/introduction/introduction-to-the-pom.html#Project_Interpolation)).

The child becomes what is shown in the figure below. As we can see it is
much more compact than what was shown in the previous chapter, and only
contains the information relevant to this project: packaging type,
groupId, artifactId and version. You will also note the addition of the
parent section that refers to the coordinate of the parent both in terms
of Maven coordinate and an optional file path.

It worth noting that not all projects built in one build have to inherit
from the same parent, and also that each child project can define
additional repositories or perform any arbitrary build steps by adding
those to their pom.xml.

    <project
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          http://maven.apache.org/xsd/maven-4.0.0.xsd"
      xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
      <modelVersion>4.0.0</modelVersion>
      <parent>
        <artifactId>parent</artifactId>
        <groupId>tychodemo</groupId>
        <version>0.0.1-SNAPSHOT</version>
        <relativePath>../tychodemo.parent/pom.xml</relativePath>
      </parent>
      <groupId>tychodemo</groupId>
      <artifactId>tychodemo.bundle</artifactId>
      <version>1.0.0-SNAPSHOT</version>
      <packaging>eclipse-plugin</packaging>
    </project>

Introducing the aggregator
==========================

When several modules need to be built together, they are typically
combined into a module that is usually referred to as an "aggregator".
Whereas parent plays the role of inheritance, the aggregator plays a
role of composition, regrouping under the same module pieces that should
be built together.

The following figure shows the complete aggregator. The modules section
lists the actual modules that will be built. It is important to note
that each entry refers to the folder on disk relative to the position of
the aggregator pom and not the project coordinate.

Another point worth noting is that modules are not built in the order
they are listed. Instead, projects are built following the topological
sort of the dependencies of the projects being built.

    <project
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          http://maven.apache.org/xsd/maven-4.0.0.xsd"
      xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
      <modelVersion>4.0.0</modelVersion>
      <groupId>tychodemo</groupId>
      <artifactId>aggregator</artifactId>
      <version>0.0.1-SNAPSHOT</version>
      <packaging>pom</packaging>

      <modules>
        <module>tychodemo.bundle</module>
        <module>tychodemo.parent</module>
      </modules>

    </project>

To summarize, we now have two folders for a total of three pom files
organized as follows: TO FILL

From there on, the addition of a bundle (or any other element) simply
consists in creating a new bundle with a pom.xml (like the one show
figure TOFILL) and adding it in the aggregator.

Finally, in this setup the build is usually started at the aggregator
level. However it is still possible to start the build from any
sub-project as long as the content depended upon is already available.

Building a feature
==================

Building features
=================

In this chapter we are showing how to build features. In addition to the
files normally required by PDE (feature.xml and build.properties), tycho
introduces a minimal pom.xml as shown below.

The most relevant parts of this pom.xml are
\<packaging\>eclipse-feature\</packaging\> which indicates that this
project is a feature, and the artifactId / version that should match the
feature id and version found in the feature.xml.

Like for plugins, the content of the feature.jar archive gets controlled
by the value of the bin.include property of the build.properties file.

    <?xml version="1.0" encoding="UTF-8"?>
    <project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                                 http://maven.apache.org/xsd/maven-4.0.0.xsd"
             xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
      <modelVersion>4.0.0</modelVersion>
      <parent>
        <artifactId>parent</artifactId>
        <groupId>tychodemo</groupId>
        <version>0.0.1-SNAPSHOT</version>
        <relativePath>../tychodemo.parent/pom.xml</relativePath>
      </parent>

      <groupId>tychodemo</groupId>
      <artifactId>tychodemo.feature</artifactId>
      <version>1.0.0-SNAPSHOT</version>

      <packaging>eclipse-feature</packaging>
    </project>

The build will produce the jar for the feature in the target folder.
However note that this archive does not include the plugins listed in
the features. The gathering of features and plugins is usually handled
by the eclipse-repository presented in a following chapter.

Aggregator vs feature
=====================

What is the difference between the aggregator and the feature?

The feature is an install time mechanism created by Eclipse to install a
set of features and plugins together. The aggregator is a build time
concept created by Maven to aggregate a set of things that need to be
built together. For example, in an aggregator it is common to find a
bundle, a corresponding bundle for tests, and the feature. However the
feature.xml itself will most likely only refer to the bundles that end
up being delivered to the final user of the application.

Building a p2 repository
========================

The creation of update site (aka p2 repository) is handled very
concisely. It only takes a pom.xml with the eclipse-repository packaging
type and a category.xml. The features that need to be made available in
the final repository are listed in the category.xml located in the same
folder than pom.xml. The category.xml files serves two purposes:

-   To list the features that need to be contained into the site.

-   To categorize the content of the repository for easier presentation
    to the user.

Note that the features referenced from the category.xml don’t have to be
those built in the current build. They can be features that are already
built and stored in p2 repositories.

For example the following category file will publish the
tychodemo.feature feature and will make it available under a category
called "Tycho Demo Category" .

    <?xml version="1.0" encoding="UTF-8"?>
    <site>
       <feature url="features/tychodemo.feature_1.0.0.qualifier.jar"
                id="tychodemo.feature" version="1.0.0.qualifier">
          <category name="tychodemo.category"/>
       </feature>
       <feature url="features/org.eclipse.rcp_0.0.0.jar"
                id="org.eclipse.rcp" version="0.0.0">
          <category name="tychodemo.category"/>
       </feature>
       <feature
          url="features/tychodemo.source.feature_1.0.0.qualifier.jar"
          id="tychodemo.source.feature" version="1.0.0.qualifier">
          <category name="tychodemo.category"/>
       </feature>
       <category-def name="tychodemo.category"
                     label="Tycho Demo Category">
          <description>
             Tycho Demo Category
          </description>
       </category-def>
    </site>

The following figure is the pom.xml file used to build a p2 repository.
Even though a category or a site don’t have an identifier, it is still
required to specify a GAV in the pom.xml. The execution of the build
will produce in the target folder a zip of the p2 repository, and the
folder target/repository/ will contained the unzipped form.

    <?xml version="1.0" encoding="UTF-8"?>
    <project
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          http://maven.apache.org/xsd/maven-4.0.0.xsd"
      xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
      <modelVersion>4.0.0</modelVersion>
      <parent>
        <artifactId>parent</artifactId>
        <groupId>tychodemo</groupId>
        <version>0.0.1-SNAPSHOT</version>
        <relativePath>../tychodemo.parent/pom.xml</relativePath>
      </parent>
      <groupId>tychodemo</groupId>
      <artifactId>tychodemo.repository</artifactId>
      <version>1.0.0-SNAPSHOT</version>
      <packaging>eclipse-repository</packaging>
    </project>

Self-contained repositories
===========================

In some cases like RCP applications, you want to make sure that the
repository you are making available is completely self-contained. That
means that for any plugin or feature available in the repository all the
other plugins and features necessary to install it are also contained in
the repository.

This is especially useful when you want to be in complete control of
where the final user of your update site is going to get the artifacts
and metadata from. For example, you may not want your user to have to
connect to eclipse.org to download the RCP plugins and features, but
only come to your own site.

The creation of such self contained repositories is supported by the
eclipse-repository packaging type, and it suffices to add the following
markup to the previous pom.xml to get the desired behaviour. TO FILL

Though using this approach is tempting to resolve dependency issues, it
needs to be clearly understood that the repositories created are much
larger than those including just your own features since it will capture
everything from your feature down to OSGi or SWT. These large
repositories have two consequences, first they are longer to download
for your user (since there is more content), but also in versions pre
eclipse 3.7 (Indigo) will result in a bigger memory footprint during the
installation.

If controlling the provenance of all the artifacts is not a necessity,
it is recommended to use repository references or composite repositories
([http://wiki.eclipse.org/Equinox/p2/Composite\_Repositories\_%28new%29](http://wiki.eclipse.org/Equinox/p2/Composite_Repositories_%28new%29))
or repository references.

Being a good repository host
============================

Now that you have created a repository, it worth understanding the
responsibilities associated with it. Indeed, by making your repository
available to others, you are providing them something that they can
start using in their products be it at build time or install time. As
such, changing the URL of your repository or removing content once it
has been made widely available can have unforeseen consequences like
breaking builds or preventing applications to install. To mitigate these
things, here are a few things that you need to be aware of:

-   Define a retention policy. For each repository you make available,
    clearly state out what is the sort of content you are making
    available

    -   final release or interim build - and how long the content will
        be here for. For example, the eclipse platform states

stable URL

Good connectivity

Controlling dependencies
========================

This chapter discuss how bundles, features and p2 IUs are made available
to the projects being build.

It also worth mentioning that in contrast with what happens in the IDE,
each project has its own dependencies and could therefore be built
against a different set of repositories, or target platform.

The source of dependencies
==========================

Tycho offers two ways to control the set of bundles, features and p2 IUs
that are accessible when the dependencies are being resolved: repository
and target platform. This section introduces how they can both be used
and analyzes their pros and cons.

Repositories
------------

Repositories, defined in the repository tag of the pom.xml (see figure
XXX TO FILL), offer a simple way to make available a set of bundles and
features to Tycho. This approach will make available all the elements
contained in the repository.

    <repository>
      <id>helios</id>
      <layout>p2</layout>
      <url>http://download.eclipse.org/releases/helios</url>
    </repository>

Advantages
:   Dealing with repositories is very simple since one just has to add a
    reference to the pom.xml to have the complete content of a
    repository be available in a build.

Disadvantages
:   Everything available in the repository is available. This means that
    if the content of the repository changes, then the build "changes"
    as well. It also means that it is "harder" to control the
    dependencies being introduced since just adding a reference from a
    bundle will suffice to get a new bundle consumed.

Target platform
---------------

A target platform identify a set of bundles and features that are made
available at build time. Target platforms are captured in target
definition files (.target files) and reuse the format defined by PDE
(Target Platform
[http://help.eclipse.org/helios/topic/org.eclipse.pde.doc.user/concepts/target.htm](http://help.eclipse.org/helios/topic/org.eclipse.pde.doc.user/concepts/target.htm)).
The following figure shows a target definition that only makes available
the content for the JDT feature.

As you can see the main difference is that in addition of listing
repositories, target definitions list a set of IUs that must be made
visible, thus limiting to the transitive closure of the IUs listed what
is available.

The following snippet shows how to use target definitions in
replacements of the repositories. TOFILL

Advantages
:   Allows you to precisely control what is made available, thus making
    it is more difficult to have undesired dependencies creep in.

        The target definition file can be used in PDE to control the set of
        bundles and features visible in the IDE when used to set the target
        platform, offering a consistent experience across the command line
        build and the IDE.

Disadvantages
:   Despite the target platform editor provided by PDE, the target
    platform file can be painful to maintain, especially to update the
    versions being used.

Build stability
===============

Whether you are using repositories or target platforms to capture your
dependencies, both approaches use p2 repositories. Therefore, for build
stability purpose, it is important to understand the contracts under
which these repositories are made available. The two key characteristics
to understand are: the kind of content made available - is it a final
release, is it intermediary build (Nightly, Integration, Milestone, in
the eclipse parlance)? - and the retention policy applied - how long
will this content stay here for? -. An example of a retention policy can
be found at TO FILL.

Unfortunately, even with that understood, the builds are still at the
mercy of other glitches like the site hosting the repository being down.
To mitigate some of those issues, it is recommended to use a repository
manager like Nexus to proxy p2 repositories, or to mirror the
repositories being used (TO FILL link to p2 mirror task).

Building a product / RCP application
====================================

In order to build an RCP application, Tycho relies the product file used
by PDE. The build of product is done using the \<eclipse-repository\>
packaging type. However in comparison to what we have seen so far,
building a product requires a little bit more configuration since a
product is usually made available as a p2 repository or as ready-to-run
archive.

Also, another noticeable difference with the typical pattern recommended
by PDE is that the product file needs to be in a project of its own
rather than being in the product that defines the application. This
gives the following file structure: TO FILL

The following pom.xml shows the creation of a product archive for a
variety of platforms specified in the parent pom. The
materialize-product reference will cause the product to be installed
locally as part of the build, and the archive-products will create the
final archives.

The result of the build is available in the target folder.

    <?xml version="1.0" encoding="UTF-8"?>
    <project
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                  http://maven.apache.org/xsd/maven-4.0.0.xsd"
      xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
      <modelVersion>4.0.0</modelVersion>
      <parent>
        <artifactId>parent</artifactId>
        <groupId>tychodemo</groupId>
        <version>0.0.1-SNAPSHOT</version>
        <relativePath>../tychodemo.parent/pom.xml</relativePath>
      </parent>
      <groupId>tychodemo</groupId>
      <artifactId>tychodemo.product</artifactId>
      <version>1.0.0-SNAPSHOT</version>
      <packaging>eclipse-repository</packaging>
      <build>
        <plugins>
          <plugin>
            <groupId>org.sonatype.tycho</groupId>
            <artifactId>tycho-p2-director-plugin</artifactId>
            <version>${tycho-version}</version>
            <executions>
              <execution>
                <!-- install the product for all configured os/ws/arch environments
                  using p2 director -->
                <id>materialize-products</id>
                <goals>
                  <goal>materialize-products</goal>
                </goals>
              </execution>
              <execution>
                <!-- (optional) create product zips (one per os/ws/arch) -->
                <id>archive-products</id>
                <goals>
                  <goal>archive-products</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
        </plugins>
      </build>
    </project>

Specifying the build environment
================================

Given that RCP applications are platform specific (e.g. Windows, Linux,
Mac), it is required to define the set of environments for which the
application needs to be built. This is typically done in the parent
pom.xml by adding the following markup. The values for os, ws and arch
are those that are supported by Eclipse ([http://TO](http://TO) FILL)

    <environments>
      <environment>
        <os>linux</os>
        <ws>gtk</ws>
        <arch>x86</arch>
      </environment>
      <environment>
        <os>linux</os>
        <ws>gtk</ws>
        <arch>x86_64</arch>
      </environment>
      <environment>
        <os>win32</os>
        <ws>win32</ws>
        <arch>x86</arch>
      </environment>
      <environment>
        <os>win32</os>
        <ws>win32</ws>
        <arch>x86_64</arch>
      </environment>
      <environment>
        <os>macosx</os>
        <ws>cocoa</ws>
        <arch>x86_64</arch>
      </environment>
    </environments>

Archive root folder
===================

By default the root folder used in the created archive is eclipse. In
order to control this, you can add the following snippet:

    <!-- (optional) customize the root folder name of the product zip -->
    <configuration>
      <products>
        <product>
          <id>tychodemo.product</id>
          <rootFolder>myRCP</rootFolder>
        </product>
      </products>
    </configuration>

Testing bundles
===============

Tycho has built-in support to execute "headless", "UI" and "SWTBot"
JUnit tests. All of these are supported through the
"eclipse-test-plugin" packaging type.

For Maven users, this is slightly different than the typical Maven way
since tests are normally stored along side with the code being tested,
whereas the Eclipse convention, followed by Tycho, separates the main
code from the tests in individual project.

Which tests are executed?
=========================

By default Tycho will execute all the tests found in any package
contained in the test plugin. This means that the smallest pom.xml for
executing headless tests is the following: TO FILL

The results of the tests are collected in the target folder TO FILL.
When the execution of the tests fails, the build is stopped. To continue
despite errors, one can specify the TO FILL parameters.

If instead you want to control which tests are being run, you can add
the following configuration attributes to the execution of the
maven-osgi-test-plugin

    <![CDATA[
    <testSuite>org.eclipse.equinox.p2.tests</testSuite>
    <testClass>org.eclipse.equinox.p2.tests.AutomatedTests</testClass>
    ]]>

Running UI tests
================

In order to execute UI tests, some additional configuration is needed to
tell Tycho to use the appropriate test harness. This can be done by
adding the following XML markup:

    <![CDATA[
    <useUIHarness>true</useUIHarness>
    ]]>

Runtime execution of the tests
==============================

By default, the runtime environment in which the tests are executed is
only composed of the bundles that are in the transitive closure of the
test bundle. Though this has the advantage of providing a more
controlled environment and thus limit perturbation that can be caused by
other bundles or test bundles, it has two drawbacks. First, fragments or
bundles providing extensions to extension points, necessary to the
execution won’t be available. Second, bugs resulting from the potential
integration will not be caught.

To address these scenarios, Tycho allows to specify additional
dependencies that must be compose the runtime environment. This can be
done by using the TO FILL attribute.

Tycho also allows to specify VM arguments and application arguments.

Finally, it is frequent for OSGi applications such as RCP to rely on the
usage of start levels. However since those are often custom to the
application, Tycho needs to be taught which are those and this can be
done by adding the following configuration:

    <![CDATA[
      <bundleStartLevel>
        <bundle>
          <id>org.eclipse.equinox.ds</id>
          <level>1</level>
          <autoStart>true</autoStart>
        </bundle>
      </bundleStartLevel>
    ]]>

Other Build Tricks
==================

This chapter discusses topics that are often discussed in the context of
Tycho such as bundle signing, running quality analysis tools.

Signing bundles
===============

The signature of bundles and features is recommended for Eclipse
applications. The signature will be checked at install time by p2.

To enable signing, the Maven plugin called maven-jar-signing can be
used. TO FILL (refernece and sample).

Generating JavaDoc
==================

Javadoc generation is covered by using the standard maven-javadoc-plugin
([http://maven.apache.org/plugins/maven-javadoc-plugin/](http://maven.apache.org/plugins/maven-javadoc-plugin/))
along with all its configuration options. The most simple example would
be to execute

    mvn javadoc:javadoc

which will create javadoc for each module in target/apidocs/.

Note that this does not cause the generation of the javadoc for
extension and extension points.

Generating source bundles
=========================

Source bundle generation is switched on using the following snippet to
be inserted in the build/plugins section of your (parent) POM:

    <!-- enable source bundle generation -->
    <plugin>
      <groupId>org.sonatype.tycho</groupId>
      <artifactId>maven-osgi-source-plugin</artifactId>
      <version>$tycho-version</version>
      <executions>
        <execution>
          <id>plugin-source</id>
          <goals>
            <goal>plugin-source</goal>
          </goals>
        </execution>
      </executions>
    </plugin>

This will create an eclipse source bundle jar along with each bundle jar
in the target/ folder of the module. By convention, the source bundle
SymbolicName is the bundle SymbolicName with ".source" appended.
Generated source bundles can then be included in a source feature which
can be created just as any other feature. Automatic source feature
generation may be added in a later version of Tycho.

Quality analysis tools
======================

Running PMD Running FindBugs Running Emma (code coverage)
