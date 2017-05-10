docker-osg-build
================

This is a Docker wrapper around `osg-build` and `osg-koji`, the two primary
tools for doing and getting information about builds in the OSG's Koji build
system. The original scripts are intended for use on RHEL 6- and 7-compatible
systems and have lots of Linux- and Red Hat-specific dependencies that aren't
available on e.g. macOS systems.

The model of these wrapper scripts is to start up a persistent Docker
container with your 'work directory' -- the directory that contains your
checkouts of OSG packages -- mounted inside the container.

These scripts are still in an experimental stage. As of 2017-05-10, they have
been tested on macOS 10.12 (Sierra).


Requirements
============
* Docker
* Your grid cert and key in `~/.globus/usercert.pem` and `~/.globus/userkey.pem`


Instructions
============

Initial image
-------------

Before using these scripts, you will need to build the Docker image that they
are based on.

Because of its experimental nature, the base Docker container for
`docker-osg-build` has been split into two images,
`matyasselmeci:osgbuilderbase` and `matyasselmeci:osgbuilder`.

`osgbuilderbase` is an image containing only the software installation of
`osg-build` and relevant software packages and dependencies. `osgbuilder` also
contains configuration and utility scripts for running OSG build commands.

To build the base image, `osgbuilderbase`, run the script named
`buildbuilderbase`.

After that, to build the main image, `osgbuilder`, run the script named
`buildbuilder`.


Starting up the image
---------------------

`docker-osg-build` uses a Docker container that runs in the background. Your
`~/.globus` directory and 'work directory,' which contains the packages you are
working with, will be mounted inside the container. You must have a valid,
current user cert and key inside `~/.globus/usercert.pem` and
`~/.globus/userkey.pem` to run `docker-osg-build`.

To create the Docker container, run:
    initbuilder <work-dir>
where `<work-dir>` is the directory under which your packages are. For example,
if you have an SVN checkout of https://vdt.cs.wisc.edu/svn/native/redhat in
`~/work/redhat`, you should use `~/work/redhat` for your `<work-dir>`. The
important thing is for your `.svn` directory to be inside the directory,
otherwise `osg-build` won't be able to find the SVN information about the
packages you're building. Similarly, if you're building packages from a Git
repo, your `.git` directory must be inside `<work-dir>`.

`initbuilder` will start a Docker container named `osgbuilder`. In case Docker
gets shut down, or the container gets turned off for some other reason, run
`docker start osgbuilder` to start the container back up.


Using the tools
---------------

This product contains two scripts, `exec-osg-koji` and `exec-osg-build` that
run the `osg-koji` and `osg-build` scripts, respectively, inside the Docker
container. You may wish to make shell aliases for them, as in:

    alias osg-build=~/docker-osg-build/exec-osg-build
    alias osg-koji=~/docker-osg-build/exec-osg-koji

The scripts will create a grid certificate (if necessary) and run `osg-koji` or
`osg-build` inside the container, with the arguments provided.

As long as the packages you are building are inside `<work-dir>` specified
above, `osg-build` will be able to read and write files inside the package
directories.


Example usage
=============

(do once):

    ~/docker-osg-build/buildbuilderbase
    ~/docker-osg-build/buildbuilder
    svn checkout https://vdt.cs.wisc.edu/svn/native/redhat ~/work/redhat
    ~/docker-osg-build/initbuilder ~/work/redhat

Subsequent operations:

    cd ~/work/redhat/osg-ce
    ~/docker-osg-build/osg-build koji --scratch --getfiles .
    ~/docker-osg-build/osg-koji list-tagged osg-3.3-el6-development

