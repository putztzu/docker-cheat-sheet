# Docker Cheat Sheet

Maintained separately by TSU based on original work by [www.github.com/wsargent](http://github.com/wsargent/docker-cheat-sheet). The reason why I decided to do a permanent fork is that much of my re-wording is stylistic rather than purely substantive. Some differences are
* Most if not all non-Docker is removed including Vagrant, MacOS (unless related to setup), more.
* I conceptualized some Docker architecture differently. The architecture is not actually different, but the way I piece things together in my mind is considerably different in some places (eg EXPOSE).

NOTE: This used to be a gist that continually expanded.  It's now a github project because it's considerably easier for other people to edit, fix and expand on Docker using Github.  Just click  [README.md](https://github.com/putztzu/docker-cheat-sheet/blob/master/README.md), and then on the "writing pen" icon on the right to edit.

* [Installation](https://github.com/putztzu/docker-cheat-sheet#installation)
* [Containers](https://github.com/putztzu/docker-cheat-sheet#containers)
* [Images](https://github.com/putztzu/docker-cheat-sheet#images)
* [Registry and Repository](https://github.com/putztzu/docker-cheat-sheet#registry--repository)
* [Dockerfile](https://github.com/putztzu/docker-cheat-sheet#dockerfile)
* [Layers](https://github.com/putztzu/docker-cheat-sheet#layers)
* [Links](https://github.com/putztzu/docker-cheat-sheet#links)
* [Volumes](https://github.com/putztzu/docker-cheat-sheet#volumes)
* [Exposing Ports](https://github.com/putztzu/docker-cheat-sheet#exposing-ports)
* [Tips](https://github.com/putztzu/docker-cheat-sheet#tips)


## Prequisites, Docker Installation
Install in a virtualization technology like Virtualbox, VMware, Hyper-V if you are not running on Linux. As of today, Docker runs only on Linux.

Whatever your distro, Docker can be installed according to directions for your distro. As of this writing docker can be installed in the regular repositories for Ubuntu, CentOS, Fedora and likely many more. OpenSUSE requires adding the Virtualization repo as described in this wiki
http://en.opensuse.org/User:Tsu2/docker

## Images
An image is a basic building block. It typically has only minimal configurations, ready for you to customize and create a running environment (a container).

## Containers

[The isolated runtime environment](http://docker.readthedocs.org/terms/container/#container-def) which is created from an image.  Containers are the app, OS, or anything you want running.  Containers are like highly featured chroots, but much more advanced features you might find in other virtualization technologies.

One User life-cycle might be 
- Launch a generic container from a generic image
- Access the console and modify the application as it's running from inside the container
- Commit your container, thereby creating a new image which now contains your configuration modifications.

Some common misconceptions it's worth correcting:

* __Containers are not transient__.  `docker run` doesn't do what you think, it actually creates new containers.
* __Containers are not limited to running a single command or process.__  You can use [supervisord](http://docs.docker.io/examples/using_supervisord/) or [runit](https://github.com/phusion/baseimage-docker).

### Lifecycle

* [`docker run`](http://docs.docker.io/reference/commandline/cli/#run) creates a container.
* [`docker stop`](http://docs.docker.io/reference/commandline/cli/#stop) stops it.
* [`docker start`](http://docs.docker.io/reference/commandline/cli/#start) will start it again.
* [`docker restart`](http://docs.docker.io/reference/commandline/cli/#restart) restarts a container.
* [`docker rm`](http://docs.docker.io/reference/commandline/cli/#rm) deletes a container.
* [`docker kill`](http://docs.docker.io/reference/commandline/cli/#kill) sends a SIGKILL to a container.  [Has issues](https://github.com/dotcloud/docker/issues/197).
* [`docker attach`](http://docs.docker.io/reference/commandline/cli/#attach) combines two containers to run as one
* [`docker wait`](http://docs.docker.io/reference/commandline/cli/#wait) blocks until container stops.

If you want to run and then interact with a container, `docker start` then `docker attach` to get in (or, as of 0.9, `nsenter`).

If you want a transient container, `docker run --rm` will remove the container after it stops.

If you want to poke around in an image, `docker run -t -i <myimage> <myshell>` to open a tty.

If you want to map a directory on the host to a docker container, `docker run -v $HOSTDIR:$DOCKERDIR` (also see Volumes section).

If you want to integrate a container with a [host process manager](http://docs.docker.io/use/host_integration/), start the daemon with `-r=false` then use `docker start -a`.

If you want to expose container ports through the host, see the [exposing ports](https://github.com/putztzu/docker-cheat-sheet#exposing-ports) section.

### Info

* [`docker ps`](http://docs.docker.io/reference/commandline/cli/#ps) shows running containers.
* [`docker inspect`](http://docs.docker.io/reference/commandline/cli/#inspect) looks at all the info on a container (including IP address).
* [`docker logs`](http://docs.docker.io/reference/commandline/cli/#logs) gets logs from container.
* [`docker events`](http://docs.docker.io/reference/commandline/cli/#events) gets events from container.
* [`docker port`](http://docs.docker.io/reference/commandline/cli/#port) shows public facing port of container.
* [`docker top`](http://docs.docker.io/reference/commandline/cli/#top) shows running processes in container.
* [`docker diff`](http://docs.docker.io/reference/commandline/cli/#diff) shows changed files in the container's FS.

`docker ps -a` shows running and stopped containers.

### Import / Export

There doesn't seem to be a way to use docker directly to import files into a container's filesystem.  The closest thing is to mount a host file or directory as a data volume and copy it from inside the container.

* [`docker cp`](http://docs.docker.io/reference/commandline/cli/#cp) copies files or folders out of a container's filesystem.
* [`docker export`](http://docs.docker.io/reference/commandline/cli/#export) turns container filesystem into tarball.

### Entering a Docker Container

The "official" way to enter a docker container while it's running is to use `nsenter`, which uses [libcontainer under the hood](http://jpetazzo.github.io/2014/03/23/lxc-attach-nsinit-nsenter-docker-0-9/).  Using an `sshd` daemon is [considered evil](http://jpetazzo.github.io/2014/06/23/docker-ssh-considered-evil/).  

Unfortunately, nsenter requires some configuration and installation. If your operating system does not include nsenter (usually in a package named util-linux or similar, although it has to be quite a recent version), the easiest way is probably to install it through docker, as described in the first of the following links:

* [Installing nsenter using docker](https://github.com/jpetazzo/nsenter)
* [How to enter a Docker container](https://blog.codecentric.de/en/2014/07/enter-docker-container/)
* [Docker debug with nsenter on boot2docker](http://blog.sequenceiq.com/blog/2014/07/05/docker-debug-with-nsenter-on-boot2docker/)

`nsenter` allows you to run any command (e.g. a shell) inside a container that's already running another command (e.g. your database or webserver). This allows you to see all mounted volumes, check on processes, log files etc. inside a running container.

The first installation method described above also installs a small wrapper script wrapping `nsenter` named `docker-enter` that makes executing a shell inside a running container as easy as `docker-enter CONTAINER` and any other command via `docker-enter CONTAINER COMMAND`.

## Images

Images are just [templates for docker containers](http://docker.readthedocs.org/reference/terms/image/).

### Lifecycle

* [`docker images`](http://docs.docker.io/reference/commandline/cli/#images) shows all images.
* [`docker import`](http://docs.docker.io/reference/commandline/cli/#import) creates an image from a tarball.
* [`docker build`](http://docs.docker.io/reference/commandline/cli/#build) creates image from Dockerfile.
* [`docker commit`](http://docs.docker.io/reference/commandline/cli/#commit) creates image from a container.
* [`docker rmi`](http://docs.docker.io/reference/commandline/cli/#rmi) removes an image.
* [`docker insert`](http://docs.docker.io/reference/commandline/cli/#insert) inserts a file from URL into image. (kind of odd, you'd think images would be immutable after create)
* [`docker load`](http://docs.docker.io/reference/commandline/cli/#load) loads an image from a tar archive as STDIN, including images and tags (as of 0.7).
* [`docker save`](http://docs.docker.io/reference/commandline/cli/#save) saves an image to a tar archive stream to STDOUT with all parent layers, tags & versions (as of 0.7).

`docker import` and `docker commit` only set up the filesystem, not Dockerfile info like CMD or ENTRYPOINT or EXPOSE.  See [bug](https://github.com/dotcloud/docker/issues/1141).

### Info

* [`docker history`](http://docs.docker.io/reference/commandline/cli/#history) shows history of image.
* [`docker tag`](http://docs.docker.io/reference/commandline/cli/#tag) tags an image to a name (local or registry).

## Registry & Repository

A repository is a *hosted* collection of tagged images that together create the file system for a container.

A registry is a *host* -- a server that stores repositories and provides an HTTP API for [managing the uploading and downloading of repositories](http://docs.docker.io/use/workingwithrepository/).

Docker.io hosts its own [index](https://index.docker.io/) to a central registry which contains a large number of repositories.

* [`docker login`](http://docs.docker.io/reference/commandline/cli/#login) to login to a registry.
* [`docker search`](http://docs.docker.io/reference/commandline/cli/#search) searches registry for image.
* [`docker pull`](http://docs.docker.io/reference/commandline/cli/#pull) pulls an image from registry to local machine.
* [`docker push`](http://docs.docker.io/reference/commandline/cli/#push) pushes an image to the registry from local machine.

## Dockerfile

[The configuration file](http://docs.docker.io/introduction/working-with-docker/#working-with-the-dockerfile). Sets up a Docker container when you run `docker build` on it.  Vastly preferable to `docker commit`.

### Instructions

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

### Tutorial

* [Flux7's Dockerfile Tutorial](http://flux7.com/blogs/docker/docker-tutorial-series-part-3-automation-is-the-word-using-dockerfile/)

### Examples

* [Examples](http://docs.docker.io/reference/builder/#dockerfile-examples)

### Best Practices

Best to look at [http://github.com/wsargent/docker-devenv](http://github.com/wsargent/docker-devenv) and the [best practices](http://crosbymichael.com/dockerfile-best-practices.html) / [take 2](http://crosbymichael.com/dockerfile-best-practices-take-2.html) for more details.

If you use [jEdit](http://jedit.org), I've put up a syntax highlighting module for [Dockerfile](https://github.com/wsargent/jedit-docker-mode) you can use.

## Layers

The [versioned filesystem](http://en.wikipedia.org/wiki/Aufs) in Docker is based on layers.  They're like [git commits or changesets for filesystems](http://docker.readthedocs.org/reference/terms/layer/).

## Links

Links are how two Docker containers can be combined (http://docs.docker.io/use/working_with_links_names/).  [Linking into Redis](http://docs.docker.io/use/working_with_links_names/#links-service-discovery-for-docker) and [Atlassian](http://blogs.atlassian.com/2013/11/docker-all-the-things-at-atlassian-automation-and-wiring/) show examples.  You can also resolve [links by hostname](http://docs.docker.io/use/working_with_links_names/#resolving-links-by-name).


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

Docker volumes are [free-floating filesystems](http://docs.docker.com/userguide/dockervolumes/).  They don't have to be connected to a particular container.

Volumes are useful in situations where you can't use links (which are TCP/IP only).  For instance, if you need to have two docker instances communicate by leaving stuff on the filesystem.

You can mount them in several docker containers at once, using `docker run -volume-from`

Because volumes are isolated filesystems, they are often used to store state from computations between transient containers.  That is, you can have a stateless and transient container run from a recipe, blow it away, and then have a second instance of the transient container pick up from where the last one left off.

See [advanced volumes](http://crosbymichael.com/advanced-docker-volumes.html) for more details.

## Exposing ports

?? Exposing ports through the host container is [fiddly but doable](http://docs.docker.io/use/port_redirection/#binding-a-port-to-an-host-interface).

Some fundemental docker architecture should be reviewed here
- Docker Containers are runtime instances based on Images.
- Docker security is like a shell around the running container
- Apps running in a Container must inform Docker from the inside what ports the app wants to accept connections.
- The "docker run" command completes the TCP/IP networking configuration by defining at least the following
-- The networking stack to be used
-- The real world IP address to be used
-- The LInux Bridging Device to be used
-- The re-mapped port to be used to avoid contention

If not defined explicitly in the "docker run" command, defaults are used. In some cases this is acceptable, but some like specifying incoming ports are best explicitly defined so are consistent and easily known (else would be random).

To tell docker your app wants to use a standard app port (don't worry if this might conflict in the real world, with "docker run" this standard port will be either mapped to the standard port or remapped if necessary)

```
EXPOSE <CONTAINERPORT>
```

This command tells docker an app running in the container wants to accept incoming connections on this standard app port. Note that the way docker works, actual networking configuration is configured in docker itself so no other networking is defined.

In the following, the container is intended to run in daemon mode (aka background like a service. The alternative is to define and immediately run a console), a localhost address is defined (warning, this won't work in a virtualized environment if docker is running in something like Virtualbox. A known issue is that you must use an actual network address that's not localhost). The real world mapped port is $HOSTPORT separated from the port defined by EXPOSE called $CONTAINERPORT. A custom name is optionally defined followed by the image the container is created from. 
```
docker run -d -p 127.0.0.1:$HOSTPORT:$CONTAINERPORT --name CONTAINER -t someimage
```

?? If you're running Docker in Virtualbox, you then need to forward the port there as well.  It can be useful to define something in Vagrantfile to expose a range of ports so that you can dynamically map them:

```
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  ...

  (49000..49900).each do |port|
    config.vm.network :forwarded_port, :host => port, :guest => port
  end

  ...
end
```

If you forget what you mapped the port to on the host container, use `docker port` to show it:

```
docker port CONTAINER $CONTAINERPORT
```

## Tips

Sources:

* [15 Docker Tips in 5 minutes](http://sssslide.com/speakerdeck.com/bmorearty/15-docker-tips-in-5-minutes)

### Last Ids

```
alias dl='docker ps -l -q'
docker run ubuntu echo hello world
docker commit `dl` helloworld
```

### Commit with command (needs Dockerfile)

```
docker commit -run='{"Cmd":["postgres", "-too -many -opts"]}' `dl` postgres
```

### Get IP address

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

### Get port mapping

```
docker inspect -f '{{range $p, $conf := .NetworkSettings.Ports}} {{$p}} -> {{(index $conf 0).HostPort}} {{end}}' <containername>
```

### Get Environment Settings

```
docker run --rm ubuntu env
```

### Delete old containers

```
docker ps -a | grep 'weeks ago' | awk '{print $1}' | xargs docker rm
```

### Delete stopped containers

```
docker rm `docker ps -a -q`
```

### Show image dependencies

```
docker images -viz | dot -Tpng -o docker.png

### My openSUSE Wiki articles
Install Docker on openSUSE
http://en.opensuse.org/User:Tsu2/docker
Access a Container Console
http://en.opensuse.org/User:Tsu2/docker-enter
Build your own Custom Container
http://en.opensuse.org/User:Tsu2/docker-build-tutorial-1


### Interesting Docker links
15 Quick Docker Tips
https://github.com/putztzu/docker-cheat-sheet.git
10 Docker Tips
Includes displaying a graphic image
http://nathanleclaire.com/blog/2014/07/12/10-docker-tips-and-tricks-that-will-make-you-sing-a-whale-song-of-joy/```
