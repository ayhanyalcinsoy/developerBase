.. _texlive-packaging:

Texlive Packages in Pardus
==========================


**Last Modified Date:** |today|

:Author: Fatih Arslan

:Version: 0.1


Current state of Texlive 2009
-----------------------------


We have actionsapi module named _texlivemodules_ that has several file
generating and populating scripts for texlive collections. This module is a one
a one copy of the Gentoo texlive eclasses. The texlive tarballs are all created
custom from the depending Gentoo ebuilds. This was done with a script that
parses the gentoo ebuilds and created a tarball for pisi packaging.

The main problem with this main workflow is the dependence to Gentoo ebuilds.
Also it and hacky and ugly way to create this packages. We dont even know what
the source of this packages are. This worklfow was also the same in Pardus 2008,
pardus 2009. We just continued it in Pardus 2011. Our whole Texlive
infrastructure was based on Gentoo. Because of this we use old Texlive 2009 based
packages in Pardus 2011.

We have a component named "Tex" that contains all TeX spesific packages and
applications. The component itself has five directories (that is the current
state and will change with the upcoming Texlive 2011). These directories are:

**addon:**      contains texlive collections and latex spesific modules

**base:**       contains basic and essential texlive collections and binaries

**doc:**        contains texlive documentations about languages

**language:**   contains texlive's langfoo collections

**tool:**       contains several tools and applications (such as kile,lyx) to work with tex itself

As you see four of the sub components are depending on Texlive itself. The addon
itself contains modules that are not Texlive spesific. Texlive collections are
mixed up with non-Texlive modules.


Texlive 2011
------------

We just use rsync to get the sources for the binary programs. Along the way we
provide a small part of the texmf tree, consisting of the configuration files of
the programs and several scripts.  (we chose to exclude several files from here
to include them in texlive-core. This is done automatically with the script

::

    rsyncupdate

This will fetch the sources for the packages texlive-bin. This package contains
compiled main texlive-packages and a small part of the texmf tree as stated
above. After syncing the packages we us the script

::

    create-bin-source-tarballs

to create a tarball for the package texlive-bin. Just be reminded that this
package itself is the main package. This is the heart of TexLive. But we also
need a functionally body that is suitable to our need in real world situations. 
We call this package texlive-core.

Now, Texlive provides packages in .tar.xz format. There are several of thousand
little,big packages like tat. Our packages are obtained by selecting a set of
TeXLive packages, uncompressing them and repacking them together. To obtain
these packages, we look at a CTAN mirror (http://www.ctan.org/tex-archive/CTAN.sites) 
in the /systems/texlive/tlnet/archive/ subdirectory.

The next problem is : how to organize these (many) packages in reasonable
packages. We follow the TeXLive organization of packages in "collections". These
collections can be seen when installing using tlmgr, the TeXLive package
manager.

We can see them  by opening for example collection-latexextra.tar.xz
(downlaod it from a TeXLive mirror) : it contains a single TLPOBJ files with a
list of dependencies, which gives the list of tar.xz archives that must be
included in this collection.( for more info on TLPOBJ files search
"TeX Live's new infrastructure" in Google)

Now the texlive-core package is not one collection, but several Texlive
collections which we are tought they're essential. The current state of the
collections included are:

::

    basic
    context
    genericrecommended
    fontsrecommended
    langczechslovak
    langdutch
    langfrench
    langgerman
    langitalian
    langpolish
    langportuguese
    langspanish
    langenglish
    latex
    latex3
    latexrecommended
    luatex
    mathextra
    metapost
    texinfo
    xetex

We have automated script that make the packages. This script download
collection, parse TLPOBJ files and creates tarball for pisi packages. The
script name is:

::

    create-tex-collection-tarball.py

You can open it and examine. The script do not just create the texlive-core
package. It also creates other collections and the merges texlive-langeextra
package. The basic workflow for the texlive-core package is:


    #. Download each collection-foo package from the essential list (as given above)
    #. Extract them to obtain the TLPOBJ file
    #. Parse the necessary modules from each collection and create a list of these modules
    #. Download each module from the previous populated list (step 3)
    #. Extract again, this time to look for runtime patterns. All modules that  contains these matches are returned
    #. Discard and remove modules that are no in the list created in step
    #. Create a final tarball to be shipped with texlive-core package

The workflow is similar for other packaging. Just examine the script. I've
seperated each package workflow in a single module. That will make it easire to
mantain for several conditions that might happen in the future.

A sidenote, collections are disjoint, that means no collections conflicts with
another. You should not face any file conflict with texlive packages. If there
exist any file conflict, than please look and fix the function that parses and
crates the dependency list along with the function that return runtime patterns.


New Component structure
-----------------------

First of all collections and tools are needed to be seperated. We will have only
three subcomponents with the following structure:

**base:**       contains the main Texlive collection ( binaries and "medium" scheme of TexLive infrastructure

**extra:**      contains extra Texlive collections modules

**addon:**      contains non-Texlive modules, such as latex-beamer, latex-hevea

**tool:**       contains several tools and applications (such as kile,lyx) to work with tex itself

Documentation is not provided anymore. It will be a seperate single tarball with
will ship in the subcomponent base in the future. The language component itself
is splited in merged into core and bin packages. That means half of the language
packages (which I believe are essential) comes with the texlivec-core package.
Other packages are merged into one package with the name texlive-langextra
