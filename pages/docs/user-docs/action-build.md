---
title: Build an Image
sidebar: user_docs
permalink: docs-build
folder: docs
---

Build is the process where we install an operating system and then configure it appropriately for a specified need. To do this we use a [Singularity recipe](/docs-recipes) file (a text file called `Singularity`) which is a recipe of how to specifically build the container. Here we will overview the usage, best practices, and quick examples.


**Getting Started**
For a detailed **walkhrough of getting started** with build, we will point you to the <a href="/quickstart">quickstart</a>. 

**Build Environment**
If you want to customize the cache, specify Docker credentials, or any custom tweaks to your build environment, see <a href="/build-environment">build environment</a>. 

**Modular Containers**
If you want to look at how to get started to making interally **modular containers**, check out <a href="http://containers-ftw.org/apps/examples/tutorials/getting-started" target="_blank">this post</a>.


{% include toc.html %}

## Bases
You have many options for the base of your build. See the linked page for each for more details, or read the [Usage](#usage) section below for a summary.

### Singularity Recipes
For a reproducible container, the recommended practice is to build by way of a Singularity recipe file. This also makes it easy to add files, environment variables, and install custom software, and still start from your base of choice (e.g., Docker). The absolute minimum required for a recipe is a base, and here are your options:


**Singularity Hub**
```
Bootstrap: shub
From: vsoch/hello-world
```

[**Docker**](/docs-docker)
```
Bootstrap: docker
From: tensorflow/tensorflow:latest
IncludeCmd: yes # Use the CMD as runscript instead of ENTRYPOINT
```

[**YUM/RHEL**](/build-yum)
```
Bootstrap: yum
OSVersion: 7
MirrorURL: http://mirror.centos.org/centos-%{OSVERSION}/%{OSVERSION}/os/$basearch/
Include: yum
```

[**Debian/Ubuntu**](/build-debootstrap)
```
Bootstrap: debootstrap
OSVersion: trusty
MirrorURL: http://us.archive.ubuntu.com/ubuntu/
```

[**Self**](/build-self)
```
Bootstrap: self
```

[**Local Image**](/build-localimage)
```
Bootstrap: localimage
From: /home/dave/starter.img 
```


## Quick Start
Too long... didn't read! If you want the quickest way to run build, here is the way to 
produce a squashfs image from a Docker base, without a hitch:


```
sudo singularity build lolcow.simg docker://godlovedc/lolcow

Docker image path: index.docker.io/godlovedc/lolcow:latest
Cache folder set to /root/.singularity/docker
Importing: base Singularity environment
Importing: /root/.singularity/docker/sha256:9fb6c798fa41e509b58bccc5c29654c3ff4648b608f5daa67c1aab6a7d02c118.tar.gz
Importing: /root/.singularity/docker/sha256:3b61febd4aefe982e0cb9c696d415137384d1a01052b50a85aae46439e15e49a.tar.gz
Importing: /root/.singularity/docker/sha256:9d99b9777eb02b8943c0e72d7a7baec5c782f8fd976825c9d3fb48b3101aacc2.tar.gz
Importing: /root/.singularity/docker/sha256:d010c8cf75d7eb5d2504d5ffa0d19696e8d745a457dd8d28ec6dd41d3763617e.tar.gz
Importing: /root/.singularity/docker/sha256:7fac07fb303e0589b9c23e6f49d5dc1ff9d6f3c8c88cabe768b430bdb47f03a9.tar.gz
Importing: /root/.singularity/docker/sha256:8e860504ff1ee5dc7953672d128ce1e4aa4d8e3716eb39fe710b849c64b20945.tar.gz
Importing: /root/.singularity/metadata/sha256:f913a03e7b5437ef6028ebb22a9a2b04362dd4919f6a19564a8669ce15bc9db5.tar.gz
Building image from sandbox: /tmp/.singularity-build.V3IfNw
Building Singularity image...
Cleaning up...
Singularity container built: lolcow.simg
vanessa@vanessa-ThinkPad-T460s:~/Documents/Dropbox/Code/singularity/singularityware.github.io/pages/docs/user-docs$ ./lolcow.simg 
 ______________________________________
/ Your motives for doing whatever good \
| deed you may have in mind will be    |
\ misinterpreted by somebody.          /
 --------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

## Usage
here is the usage. We will break it up into sections so we can talk about each one.

```bash
$ singularity build

USAGE: singularity [...] build <container path> <BUILD SPEC TARGET>
The build command compiles a container per a recipe (definition file) or based
on a URI, location, or archive.

CONTAINER PATH:
    When Singularity builds the container, the output format can be one of
    multiple formats:

        default:    The compressed Singularity read only image format (default)
        sandbox:    This is a read-write container within a directory structure
        writable:   Legacy writable image format
    
    note: A common workflow is to use the "sandbox" mode for development of the
    container, and then build it as a default Singularity image  when done. 
    This format can not be modified.

BUILD SPEC TARGET:
    The build spec target is a definition, local image, archive, or URI that
    can be used to create a Singularity container. Several different
    local target formats exist:

        def file  : This is a recipe for building a container (examples below)
        directory:  A directory structure containing a (ch)root file system
        image:      A local image on your machine (will convert to squashfs if
                    it is legacy or writable format)
        tar/tar.gz: An archive file which contains the above directory format
                    (must have .tar in the filename!)

    Targets can also be remote and defined by a URI of the following formats:

        shub://     Build from a Singularity registry (Singularity Hub default)
        docker://   This points to a Docker registry (Docker Hub default)
```

These "build spec options" refer to the [recipe bases](/docs-recipes#recipes) options that you have. 
Generally you are going to be building into a container or folder from some base that is local (an image or directory) or
from an external endpoint (e.g., docker). Next we can look at create options:

## Development (writable) or Production (immutable)?
When you build your container, you probably want a writable container for development, and a nice
immutable one for your completed, production image. Here are the options under the usage (help) for build:

```
CREATE OPTIONS:
    -s|--sandbox    Build a sandbox rather then a read only compressed image
    -w|--writable   Build a writable image (warning: deprecated due to sparse
                    file image corruption issues)
    -f|-F|--force   Force a rebootstrap of a base OS (note: this does not 
                    delete what is currently in the image, just causes the core 
                    to be reinstalled)
    -T|--notest     Bootstrap without running tests in %test section
    -s|--section    Only run a given section within the bootstrap (setup,
                    post, files, environment, test, labels, none)

```

You can think of the different options as follows:

**create writable development folder**
```
sudo singularity build --writable container/ Singularity
```

**create writable development ext3 image**
```
sudo singularity build --writable container.img Singularity
```

**create production immutable image**
```
sudo singularity build container.simg Singularity
```

For details about the Singularity flow and use cases, see our [quickstart](/quickstart).

## Checks
Next, we will look at checks. Checks haven't been robustly developed (yet!) but offer
an easy way for an admin to define a security (or any other kind of check) to be run on demand
for a Singularity image. They are defined (and run) via different tags.

```
CHECKS OPTIONS:
    -c|--checks    enable checks
    -t|--tag       specify a check tag (not default)
    -l|--low       Specify low threshold (all checks, default) 
    -m|--med       Perform medium and high checks
    -h|--high      Perform only checks at level high

See singularity check --help for available tags
```


## Examples
Let's quickly look at some definition file examples. These come straight from `singularity help build`. First, here
is a recipe for build that serves only to describe the sections.

```
    %setup
        echo "This is a scriptlet that will be executed on the host, as root, after"
        echo "the container has been bootstrapped. To install things into the container"
        echo "reference the file system location with $SINGULARITY_BUILDROOT"

    %post
        echo "This scriptlet section will be executed from within the container after"
        echo "the bootstrap/base has been created and setup"

    %test
        echo "Define any test commands that should be executed after container has been"
        echo "built. This scriptlet will be executed from within the running container"
        echo "as the root user. Pay attention to the exit/return value of this scriptlet"
        echo "as any non-zero exit code will be assumed as failure"
        exit 0

    %runscript
        echo "Define actions for the container to be executed with the run command or"
        echo "when container is executed."

    %startscript
        echo "Define actions for container to perform when started as an instance."

    %labels
        HELLO MOTO
        KEY VALUE

    %files
        /path/on/host/file.txt /path/on/container/file.txt 
        relative_file.txt /path/on/container/relative_file.txt

    %environment 
        LUKE=goodguy
        VADER=badguy
        HAN=someguy
        export HAN VADER LUKE

```

Now let's look at a recipe that defines different <a href="https://containers-ftw.github.io/apps/" target="_blank">SCI-F apps</a>


```
    %appinstall app1
        echo "These are steps to install an app using the SCI-F format"

    %appenv app1
        APP1VAR=app1value
        export APP1VAR

    %apphelp app1
        This is a help doc for running app1

    %apprun app1
        echo "this is a runscript for app1"

    %applabels app1
        AUTHOR tolkien

    %appfiles app1
        /file/on/host/foo.txt /file/in/contianer/foo.txt

    %appsetup app1
        echo "a %setup section (see above) for apps"

    %apptest app1
        echo "some test for an app" 
```

## Building
We have a recipe, great! Now how do we build it? In many ways:

```
    Build a compressed image from a Singularity recipe file:
        $ singularity build /tmp/debian0.simg /path/to/debian.def

    Build a base compressed image from Docker Hub:
        $ singularity build /tmp/debian1.simg docker://debian:latest

    Build a base sandbox from DockerHub, make changes to it, then build image
        $ singularity build --sandbox /tmp/debian docker://debian:latest
        $ singularity exec --writable /tmp/debian apt-get install python
        $ singularity build /tmp/debian2.simg /tmp/debian
```

## More Details
 - For a detailed **walkhrough of getting started** with build, we will point you to the <a href="/quickstart">quickstart</a>. 
 - If you want to customize the cache, specify Docker credentials, or any custom tweaks to your build environment, see <a href="/build-environment">build environment</a>. 
 - If you want to look at how to get started to making interally **modular containers**, check out <a href="http://containers-ftw.org/apps/examples/tutorials/getting-started" target="_blank">this post</a>.


## Support
Have a question, or need further information? <a href="/support">Reach out to us.</a>

<script>
// Without this, pagination links to exec under repeated build section
$(document).ready(function() {
    $(".next-button").closest('a').attr('href', '/docs-recipes')
})
</script>