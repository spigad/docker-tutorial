We’ll cover the following basic instructions to get you started:

* **FROM** - every Dockerfile starts with FROM, with the introduction of multi-stage builds as of version 17.05, you can have more than one FROM instruction in one Dockerfile.
* **COPY** - these two are often confused, so I’ll explain the difference.
* **ENV** - well, setting environment variables is pretty important.
* **RUN** - let’s run commands.
* **VOLUME* - another source of confusion, what’s the difference between Dockerfile VOLUME and container volumes?
* **USER** - when root is too mainstream.
* **WORKDIR** - set the working directory.
* **EXPOSE** - get your ports right.
* **ONBUILD** - give more flexibility to your team and clients.


### FROM
For beginners it’s enough to understand that every Dockerfile must start with the FROM instruction in the form of FROM <image>[:tag]. This will set the base image for your Dockerfile, which means that subsequent instructions will be applied to this base image.

The tag value is optional, if you don’t specify the tag Docker will use the tag latest and will try and use or pull the latest version of the base image during build.

On the little bit more advanced side, let’s note the following:

There is one instruction that you can put before FROM into your Dockerfile. This instruction is ARG. ARG is used to specify arguments for the docker build command with the --build-arg <varname>=<value> flag.
You can have more than one FROM instructions in your Dockerfile. You will want to use this feature, for example, when you use one base image to build your app and another base image to run it.
It’s called a multi-stage build and you can read about it here.

This is why every section that starts with FROM in your Dockerfile is called a build stage (even in the simple case of having only one FROM instruction). You can specify the name of the build stage in the form FROM <image>[:tag] [AS <name>].

## COPY vs ADD
Both ADD and COPY are designed to add directories and files to your Docker image in the form of ADD <src>... <dest> or COPY <src>... <dest>. Most resources, including myself, suggest to use COPY.

The reason behind this is that ADD has extra features compared to COPY that make ADD more unpredictable and a bit over-designed. ADD can pull files from url sources, which COPY cannot. ADD can also extract compressed files assuming it can recognize and handle the format. You cannot extract archives with COPY.

The ADD instruction was added to Docker first, and COPY was added later to provide a straightforward, rock solid solution for copying files and directories into your container’s file system.

If you want to pull files from the web into your image I would suggest to use RUN and curl and uncompress your files with RUN and commands you would use on the command line.

## ENV
ENV is used to define environment variables. The interesting thing about ENV is that it does two things:

You can use it to define environment variables that will be available in your container. So when you build an image and start up a container with that image you’ll find that the environment variable is available and is set to the value you specified in the Dockerfile.
You can use the variables that you specify by ENV in the Dockerfile itself. So in subsequent instructions the environment variable will be available.

### RUN
RUN will execute commands, so it’s one of the most-used instructions. I would like to highlight two points:

You’ll use a lot of apt-get type of commands to add new packages to your image. It’s always advisable to put apt-get update and apt-get install commands on the same line. This is important because of layer caching. Having these on two separate lines would mean that if you add a new package to your install list, the layer with apt-get update will not be invalidated in the layer cache and you might end up in a mess. Read more here.
RUN has two forms; RUN <command> (called shell form) and RUN ["executable", "param1", "param2"] called exec form. Please note that RUN <command> will invoke a shell automatically (/bin/sh -c by default), while the exec form will not invoke a command shell. If you want to tackle a problem around this read here.

### VOLUME
This is where I found Docker documentation not so easy to follow. So let me put it in plain English.

You can use the VOLUME instruction in a Dockerfile to tell Docker that the stuff you store in that specific directory should be stored on the host file system not in the container file system. This implies that stuff stored in the volume will persist and be available also after you destroy the container.

In other words it is best practice to crate a volume for your data files, database files, or any file or directory that your users will change when they use your application.

The data stored in the volume will remain on the host machine even if you stop the container and remove the container with docker rm. (The volume will be removed on exit if you start the container with docker run --rm, though.)

You can also share these volumes between containers with docker run --volumes-from.

You can inspect your volumes with the docker volume ls and docker volume inspect commands.

You can also have a look inside your volumes by navigating to Docker volumes in your file system. On Linux you can go to /var/lib/docker/volumes pick the id of the volume and list it as a directory. You can find out the id of the container and thus the volume by running docker inspect on your container.

On Mac, you’ll not be able to access /var/lib/docker/volumes so easily. If you run screen ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/tty on your Mac, you get a terminal where you can navigate as if you were using a Linux machine.

The difference between the VOLUME instruction in Dockerfile and starting your container with docker run -v ... is this: VOLUME in Dockerfile will create a new empty directory for your files under the standard Docker structure, i.e. /var/lib/docker/volumes. docker run -v ... can do more, you can mount existing directories from your host file system into your container and you can also specify the path of the directory on the host.

Now you may think that docker run -v ... is the better option, but keep in mind that mounting an existing directory assumes that the directory exists on the host, which may give you portability issues.

This topic is a good candidate for another detailed post and video, so let’s move on for now.

USER
Don’t run your stuff as root, be humble, use the USER instruction to specify the user. This user will be used to run any subsequent RUN, CMD AND ENDPOINT instructions in your Dockerfile.

WORKDIR
A very convenient way to define the working directory, it will be used with subsequent RUN, CMD, ENTRYPOINT, COPY and ADD instructions. You can specify WORKDIR multiple times in a Dockerfile.

If the directory does not exists, Docker will create it for you.

EXPOSE
An important instruction to inform your users about the ports your application is listening on. EXPOSE will not publish the port, you need to use docker run -p... to do that when you start the container.

CMD and ENTRYPOINT
CMD is the instruction to specify what component is to be run by your image with arguments in the following form: CMD [“executable”, “param1”, “param2”…].

You can override CMD when you’re starting up your container by specifying your command after the image name like this: $ docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...].

You can only specify one CMD in a Dockerfile (OK, physically you can specify more than one, but only the last one will be used).

It is good practice to specify a CMD even if you are developing a generic container, in this case an interactive shell is a good CMD entry. So you do CMD ["python"] or CMD [“php”, “-a”] to give your users something to work with.

So what’s the deal with ENTRYPOINT? When you specify an entry point, your image will work a bit differently. You use ENTRYPOINT as the main executable of your image. In this case whatever you specify in CMD will be added to ENTRYPOINT as parameters.

ENTRYPOINT ["git"]
CMD ["--help"]
This way you can build Docker images that mimic the behavior of the main executable you specify in ENTRYPOINT.

ONBUILD
This is so nice. You can specify instructions with ONBUILD that will be executed when your image is used as the base image of another Dockerfile. :)

This is useful when you want to create a generic base image to be used in different variations by many Dockerfiles, or in many projects or by many parties.

So you do not need to add the specific stuff immediately, like you don’t need to copy the source code or config files in the base image. How could you even do that, when these things will be available only later?

So what you do instead is to add ONBUILD instructions. So you can do something like this:

ONBUILD COPY . /usr/src/app
ONBUILD RUN /usr/src/app/mybuild.sh
ONBUILD instructions will be executed right after the FROM instruction in the downstram Dockerfile


