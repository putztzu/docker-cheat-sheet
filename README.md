# Docker Cheat Sheet
An extraordinarily verbose compilation of useful Docker commands, where they are used and examples
## Table of Contents
* [Installation](https://github.com/putztzu/docker-cheat-sheet#installation)
* [Images](https://github.com/putztzu/docker-cheat-sheet#images)
* [Containers](https://github.com/putztzu/docker-cheat-sheet#containers)
* [Registry and Repository](https://github.com/putztzu/docker-cheat-sheet#registry--repository)
* [Dockerfile](https://github.com/putztzu/docker-cheat-sheet#dockerfile)
* [Layers](https://github.com/putztzu/docker-cheat-sheet#layers)
* [Links](https://github.com/putztzu/docker-cheat-sheet#links)
* [Volumes](https://github.com/putztzu/docker-cheat-sheet#volumes)
* [Exposing Ports](https://github.com/putztzu/docker-cheat-sheet#exposing-ports)
* [Tips](https://github.com/putztzu/docker-cheat-sheet#tips)
* [Misc Useful Tips](https://github.com/putztzu/docker-cheat-sheet#misc-useful-tips)
* [My openSUSE Wiki articles](https://github.com/putztzu/docker-cheat-sheet#my-opensuse-wiki-articles)
* [Interesting Docker links](https://github.com/putztzu/docker-cheat-sheet#interesting-docker-links)
* [Credit to Original Work](https://github.com/putztzu/docker-cheat-sheet#credit-to-original-work)


## Prerequisites, Docker Installation
If you are running on Windows, Solaris, BSD or some other non Linux OS, run Docker in a virtualization technology like Virtualbox, VMware, Hyper-V. If running on Windows and you are only mildly familiar with virtualization, [boot2docker](http://boot2docker.io/) is a project that installs Virtualbox, tinycorelinux(Debian based) and docker with a number of desirable apps like ssh at once. If you are experienced with any virtualization or already have some kind of virtualization installed, you can create a Guest and install Docker in that Guest in an ordinary way.

Whatever distro you prefer, Docker should be installed according to directions for your distro. As of this writing docker can be installed using the regular repositories for Ubuntu, CentOS, Fedora and likely many more. Docker Install instructions can be found on this [Docker documentation pagee](https://docs.docker.com/installation/#installation). A better guide for installing docker on  OpenSUSE is here:

http://en.opensuse.org/User:Tsu2/docker

## Images
The [Docker reference](http://docker.readthedocs.org/reference/terms/image/).<br />
An image is a basic building block. Public images typically have only minimal configurations, ready for you to customize and create a running environment (a container).

#### Common Image management commands

* [`docker images`](http://docs.docker.io/reference/commandline/cli/#images) displays all images in the local repository.
* [`docker import`](http://docs.docker.io/reference/commandline/cli/#import) extracts an image from a tarball file.
* [`docker build`](http://docs.docker.io/reference/commandline/cli/#build) creates image from Dockerfile.
* [`docker commit`](http://docs.docker.io/reference/commandline/cli/#commit) creates image from a container.
* [`docker rmi`](http://docs.docker.io/reference/commandline/cli/#rmi) removes an image from the local repository.
* [`docker insert`](http://docs.docker.io/reference/commandline/cli/#insert) inserts a file from URL into image. (kind of odd, you'd think images would be immutable after create)
* [`docker load`](http://docs.docker.io/reference/commandline/cli/#load) Using STDIN, loads an image with its tags from a tar archive
* [`docker save`](http://docs.docker.io/reference/commandline/cli/#save) saves an image to a tar archive by streaming to STDOUT with all parent layers, tags & versions.
* [docker history](https://docs.docker.com/reference/commandline/cli/#history) displays the steps used to create and modify the image. Important to understand among things the underlying distro used, TCP/IP ports presented to docker (You then need to match those ports with your "docker run" command)


## Containers

A Container is the [isolated runtime environment](http://docker.readthedocs.org/terms/container/#container-def) which is created from an image.  Within a container there can be an app, OS, or anything you want running.  Containers are like highly featured chroots, but with much more advanced features you might find in other virtualization technologies.

A typical container life-cycle might be 
- Launch a generic container from a generic image
- Access the console and modify the application as it's running from inside the container
- Commit your container, thereby creating a new image which now contains your configuration modifications. When you stop and next time start your new container, it will retain your changes.

#### Common Container management commands

* [`docker run`](http://docs.docker.io/reference/commandline/cli/#run) creates a container.
* [`docker stop`](http://docs.docker.io/reference/commandline/cli/#stop) stops the container.
* [`docker start`](http://docs.docker.io/reference/commandline/cli/#start) stops and restarts the container.
* [`docker restart`](http://docs.docker.io/reference/commandline/cli/#restart) restarts a container.
* [`docker rm`](http://docs.docker.io/reference/commandline/cli/#rm) deletes a container.
* [`docker kill`](http://docs.docker.io/reference/commandline/cli/#kill) sends a SIGKILL to a container. 
* [`docker attach`](http://docs.docker.io/reference/commandline/cli/#attach) allows view or interact with a running container 
* [`docker wait`](http://docs.docker.io/reference/commandline/cli/#wait) blocks until container stops.

If you want a transient container, `docker run --rm` will remove the container after it stops.

Containers generally are created to run either as a background process (aka "service") or as an interactive "foreground" (normal app). Another way to compare the two is that a background "daemon" container continues to run without a logged in User. A foreground "nomral app" container instantiates with immediate access to a running console, and when the User exits/quits the console the container also stops.
* An example creating a daemon/service/background "detached" container specifying an optional custom container name<br />
```
'docker run -d --name containername imagename'
```
* An example creating a foreground/normal app, specifying an interactive tty bash console.<br />
```
'docker run -it --name containername imagename /bin/bash'
```
If you want to map a directory(often described as "folder sharing") on the host to a docker container,<br />
```
 `docker run -v $HOSTDIR:$DOCKERDIR` (also see Volumes section).
```
#### Incoming Network Connections

If you want to allow incoming network connections to the app in your container, see the [exposing ports](https://github.com/putztzu/docker-cheat-sheet#exposing-ports) section.


#### Configuring a Container to auto start on boot

This is described in the Docker documentation as [host process manager integration](http://docs.docker.io/use/host_integration/). In plain words, after you have a tested running container, you invoke it with "docker start -a" and modify the docker daemon (ie if using systemd, modify the Unitfile) to start with "-r=false." You can find a sample Unitfile script in the referenced Docker documentation. 
#### Info

* [`docker ps`](http://docs.docker.io/reference/commandline/cli/#ps) lists running containers.
* [`docker ps -a`](http://docs.docker.io/reference/commandline/cli/#ps) lists all containers, both  running and stopped.
* [`docker inspect`](http://docs.docker.io/reference/commandline/cli/#inspect) queries a container or image for low level information
* [`docker logs`](http://docs.docker.io/reference/commandline/cli/#logs) gets logs from container.
* [`docker events`](http://docs.docker.io/reference/commandline/cli/#events) gets events from container.
* [`docker port`](http://docs.docker.io/reference/commandline/cli/#port) identifies the external port number for the specified container port (See EXPOSE)
* [`docker top`](http://docs.docker.io/reference/commandline/cli/#top) shows running processes in container.
* [`docker diff`](http://docs.docker.io/reference/commandline/cli/#diff) shows changed files in the container's file system.


#### Import / Export

Note - this section under review and will be modified to describe the ADD dockerfile command)

* [`docker cp`](http://docs.docker.io/reference/commandline/cli/#cp) copies files or folders out of a container's filesystem.
* [`docker export`](http://docs.docker.io/reference/commandline/cli/#export) turns container filesystem into tarball.

#### Entering a Docker Container

The most recommended way to enter a docker container while it's running is to use [nsenter](http://jpetazzo.github.io/2014/03/23/lxc-attach-nsinit-nsenter-docker-0-9/).  Using an `sshd` daemon is the official documentation but [considered evil](http://jpetazzo.github.io/2014/06/23/docker-ssh-considered-evil/). Note that sshd requires installing and configuring sshd, exposing a network stack and remoting in using TCP/IP sockets. Aside from the complexity setting that all up, it's also not always possible. Nsenter uses unix sockets minimizing dependencies and complexity, so in theory should be less complex and more universally possible.

* I recommend following my [wiki article](http://en.opensuse.org/User:Tsu2/docker-enter) as simplest but you can also follow others who have written about nsenter
* [Installing nsenter using docker](https://github.com/jpetazzo/nsenter)
* [How to enter a Docker container](https://blog.codecentric.de/en/2014/07/enter-docker-container/)
* [Docker debug with nsenter on boot2docker](http://blog.sequenceiq.com/blog/2014/07/05/docker-debug-with-nsenter-on-boot2docker/)

`nsenter` allows you to run any command from a console within the running container. If the container doesn't already have the pre-isntalled app or tool, you can often install the app on the fly. If that's not possible, then you may need to build your own image and pre-install the tool or app. 

The nsenter documentation you follow should describe how to use a command "docker-enter" which is a wrapper for nsenter with a series of commonly desired command attributes (see the official nsentern documentation for more details). You can also append a command to the end of the docker-enter command if you wish to execute the command by default (not typically necessary).

#### Info

* [`docker history`](http://docs.docker.io/reference/commandline/cli/#history) displays how the image has been built.
* [`docker tag`](http://docs.docker.io/reference/commandline/cli/#tag) assigns a custom name/alias to the image (local or registry).

## Registry & Repository

Note: This section under review<br />
A repository is a *hosted* collection of tagged images. The public docker repository can be searched with "docker search"
When a copy of the image is downloaded and stored locally for personal use, you have a local repository.
You can also specify other private repositories.

A registry is a *host* -- a server that stores repositories and provides an HTTP API for [managing the uploading and downloading of repositories](http://docs.docker.io/use/workingwithrepository/).

Docker.io hosts its own [index](https://index.docker.io/) to a central registry which contains a large number of repositories.

* [`docker login`](http://docs.docker.io/reference/commandline/cli/#login) to login to a registry.
* [`docker search`](http://docs.docker.io/reference/commandline/cli/#search) searches registry for image.
* [`docker pull`](http://docs.docker.io/reference/commandline/cli/#pull) pulls an image from registry to local machine.
* [`docker push`](http://docs.docker.io/reference/commandline/cli/#push) pushes an image to the registry from local machine.

## Dockerfile

Dockerfile (exactly as shown including capitalized "D") is [the configuration file](http://docs.docker.io/introduction/working-with-docker/#working-with-the-dockerfile) used to build an image. Can be thought of as the "Install file" that defines the steps used to build an image. Typically it will start with a base image defined by FROM, identify its creator/maintainer with MAINTAINER, a sequence of RUN steps, define some app ports with EXPOSE and end by executing a CMD or ENTRYPOINT to start an application

#### Some Common Dockerfile Elements

* [FROM](http://docs.docker.io/reference/builder/#from)
* [MAINTAINER](http://docs.docker.io/reference/builder/#maintainer)
* [RUN](http://docs.docker.io/reference/builder/#run)
* [CMD](http://docs.docker.io/reference/builder/#cmd)
* [EXPOSE](http://docs.docker.io/reference/builder/#expose)
* [ENV](http://docs.docker.io/reference/builder/#env)
* [ADD](http://docs.docker.io/reference/builder/#add)
* [ENTRYPOINT](http://docs.docker.io/reference/builder/#entrypoint)
* [VOLUME](http://docs.docker.io/reference/builder/#volume)
* [USER](http://docs.docker.io/reference/builder/#user)
* [WORKDIR](http://docs.docker.io/reference/builder/#workdir)
* [ONBUILD](http://docs.docker.io/reference/builder/#onbuild)

#### Tutorials

* [Flux7's Dockerfile Tutorial](http://flux7.com/blogs/docker/docker-tutorial-series-part-3-automation-is-the-word-using-dockerfile/)

#### Docker Documentation Examples

* Note that the official Docker [Examples](http://docs.docker.io/reference/builder/#dockerfile-examples) describe techniques and methods, and not simply ways to implement a solution described by the example title. The technique and method is hidden and not described obviously, it's up to the Student to identify and extract those lessons.


## Layers

Note: This section under review

The [versioned filesystem](http://en.wikipedia.org/wiki/Aufs) in Docker is based on layers.  They're like [git commits or changesets for filesystems](http://docker.readthedocs.org/reference/terms/layer/).

## Links

Note: This section under review

Links are how two Docker containers can be combined (http://docs.docker.io/use/working_with_links_names/).  [Linking into Redis](http://docs.docker.io/use/working_with_links_names/#links-service-discovery-for-docker) and [Atlassian](http://blogs.atlassian.com/2013/11/docker-all-the-things-at-atlassian-automation-and-wiring/) show examples.  You can also resolve [links by hostname](http://docs.docker.io/use/working_with_links_names/#resolving-links-by-name).


UPDATE NOTE:
From here to the end of this "Links" section, the following example is from the original pre-fork cheat sheet and has not been verified

NOTE: If you want containers to ONLY communicate with each other through links, start the docker daemon with `-icc=false` to disable inter process communication.

If you have a container with the name CONTAINER (specified by `docker run --name CONTAINER`) and in the Dockerfile, it has an exposed port:

```
EXPOSE 1337
```

Then if we create another container called LINKED like so:

```
docker run -d --link CONTAINER:ALIAS --name LINKED user/wordpress
```

Then the exposed ports and aliases of CONTAINER will show up in LINKED with the following environment variables:

```
$ALIAS_PORT_1337_TCP_PORT
$ALIAS_PORT_1337_TCP_ADDR
```

And you can connect to it that way.

To delete links, use `docker rm --link `.

## Volumes

Note: This secion under review

Docker volumes are [free-floating filesystems](http://docs.docker.com/userguide/dockervolumes/).  They don't have to be connected to a particular container.

Volumes are useful in situations where you can't use links (which are TCP/IP only).  For instance, if you need to have two docker instances communicate by leaving stuff on the filesystem.

You can mount them in several docker containers at once, using `docker run -volume-from`

Because volumes are isolated filesystems, they are often used to store state from computations between transient containers.  That is, you can have a stateless and transient container run from a recipe, blow it away, and then have a second instance of the transient container pick up from where the last one left off.

See [advanced volumes](http://crosbymichael.com/advanced-docker-volumes.html) for more details.

## Exposing ports
aka Setting up Incoming Network Connections<br />
[The official Docker documentation](http://docs.docker.io/use/port_redirection/#binding-a-port-to-an-host-interface).<br />

Some fundemental docker architecture should be reviewed here
- Docker Containers are runtime instances based on Images.
- Docker security is like a shell around the running container
- Apps running in a Container must inform Docker from the inside what ports the app wants to accept connections.
- The "docker run" command completes the TCP/IP networking configuration from the outside by defining at least the following<br />
-- The networking stack to be used<br />
-- The real world IP address to be used<br />
-- The LInux Bridging Device to be used<br />
-- The re-mapped port to be used to avoid contention

If any networking parameter is not defined explicitly in the "docker run" command, defaults are applied. In some cases this is acceptable, but some configs like specifying incoming ports are best explicitly defined so are consistent and easily known (else would be random).

To tell docker your app wants to use a standard app port (don't worry if this might conflict in the real world, with "docker run" this standard port will be either mapped to the standard port or remapped if necessary)

```
EXPOSE <CONTAINERPORT>
```

This command tells docker an app running inside the container wants to accept incoming connections on this standard app port. Note that the way docker works, actual networking configuration is configured in docker itself so no other networking like ip addressing is defined.

In the following example, the container is intended to run in daemon mode (aka background like a service. The alternative is to define and immediately run a console), a localhost address is defined (warning, this won't work in a virtualized environment if docker is running in something like Virtualbox. A known issue is that you must use an actual network address that's not localhost). The real world mapped port is $HOSTPORT separated from the port defined by EXPOSE called $CONTAINERPORT. A custom name is optionally defined followed by the image the container is created from. 
```
docker run -d -p 127.0.0.1:$HOSTPORT:$CONTAINERPORT --name CONTAINER -t someimage
```

If you forget what you mapped the port to on the host container, use `docker port` to display it:

```
docker port CONTAINER $CONTAINERPORT
```

## Tips
### Reference an image or container with only a couple taps
Probably the most tedious part of Docker is typing all those odd names and long ID strings. Supercharge your work with the following techniques!
#### Use Custom names, make them descriptive and short
If you can remember what they mean, a letter or number might be enough
#### Just type the first couple or few numbers in the ID

The actual containerid or imageid is over 40 characters long (I haven't counted what the actual number is. Listing the container or image (eg docker ps or docker images) displays a truncated string but you can type even fewer digits! As an example, if you have fewer than 10 images in your local repository, typing only the first two digits is already likely unique enough to reference a specific image and that is all that's necessary,

The following example removes an image with an imageid that starts with `a1b2c3d4e5f5` <br />
```
docker rmi a1
```
Note: The rest of this section is under review

Source for the next tips:

* [15 Docker Tips in 5 minutes](http://sssslide.com/speakerdeck.com/bmorearty/15-docker-tips-in-5-minutes)

#### Last Ids

```
alias dl='docker ps -l -q'
docker run ubuntu echo hello world
docker commit `dl` helloworld
```
#### Commit with command (needs Dockerfile)

```
docker commit -run='{"Cmd":["postgres", "-too -many -opts"]}' `dl` postgres
```
#### Get IP address

```
docker inspect `dl` | grep IPAddress | cut -d '"' -f 4
```
or

```
wget http://stedolan.github.io/jq/download/source/jq-1.3.tar.gz
tar xzvf jq-1.3.tar.gz
cd jq-1.3
./configure && make && sudo make install
docker inspect `dl` | jq -r '.[0].NetworkSettings.IPAddress'
```
or (this is unverified)
```
docker inspect -f '{{ .NetworkSettings.IPAddress }}' <container_name>
```
#### Get port mapping
```
docker inspect -f '{{range $p, $conf := .NetworkSettings.Ports}} {{$p}} -> {{(index $conf 0).HostPort}} {{end}}' <containername>
```
#### Get Environment Settings
```
docker run --rm ubuntu env
```
#### Delete old containers
```
docker ps -a | grep 'weeks ago' | awk '{print $1}' | xargs docker rm
```
#### Delete stopped containers

```
docker rm `docker ps -a -q`
```
#### Show image dependencies
```
docker images -viz | dot -Tpng -o docker.png
```
### Misc Useful tips

Containers are not limited to running a single command or process. You can use [supervisord](http://docs.docker.io/examples/using_supervisord/) or [runit](https://github.com/phusion/baseimage-docker).

If you use [jEdit](http://jedit.org), wsargent has put up a syntax highlighting module for [Dockerfile](https://github.com/wsargent/jedit-docker-mode) you can use.


## My openSUSE Wiki articles

Install Docker on openSUSEt<br />
http://en.opensuse.org/User:Tsu2/dockert<br />
Access a Container Consolet<br />
http://en.opensuse.org/User:Tsu2/docker-entert<br />
Build your own Custom Containert<br />
http://en.opensuse.org/User:Tsu2/docker-build-tutorial-1


## Interesting Docker links

15 Quick Docker Tips<br />
https://github.com/putztzu/docker-cheat-sheet.git<br />
10 Docker Tipst<br />
Includes displaying a graphic imaget<br />
http://nathanleclaire.com/blog/2014/07/12/10-docker-tips-and-tricks-that-will-make-you-sing-a-whale-song-of-joy/``

## Credit to Original Work

Based on original work by [www.github.com/wsargent](http://github.com/wsargent/docker-cheat-sheet). 
`
