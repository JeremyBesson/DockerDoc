# DockerDoc

## Dockerfile syntax

**Extract of "DevOps with Kubernetes" (Published by Packt Publishing Ltd.
   Hideto Saito, Hui-Chuan Chloe Lee, Cheng-Yang Wu)**
   
   
The building blocks of a  Dockerfile are a dozen or more directives;
most of them are a counterpart of the functions of docker run/create flags.
Here we list the most essential ones:
- FROM:
```Dockerfile
FROM <IMAGE>[:TAG|[@DIGEST]
```
> This is to tell the Docker daemon which image the current Dockerfile is
> based on. It's also the one and only instruction
> that must be in a Dockerfile , which means that you can have a Dockerfile that
> contains only one line. Like all the other image-relevant commands,
> the tag defaults to the latest if unspecified.
- RUN:
```Dockerfile
  RUN <commands>
  RUN ["executable", "params", "more params"]
```
> The `RUN` instruction runs one line of a command at the current cache layer,
> and commits out the outcome. The main discrepancy between the two forms is in how
> the command is executed.
>
> The first one is called shell form , which actually executes commands in the form of
> `/bin/sh -c <commands>;` the other form is called exec form , and it treats
> the command with exec directly.
>
> Using the shell form is similar to writing shell scripts,
> thus concatenating multiple commands by shell operators and line continuation,
> condition tests, or variable, substitutions are totally valid. But bear
> in mind that commands are not processed by bash but sh.
>
> The exec form is parsed as a JSON array, which means that you have to wrap texts
> with double quotes and escape reserved characters. Besides, as the command is
> not processed by any shell, the shell variables in the array will not be evaluated.
> On the other hand, if the shell doesn't exist in the base image, you can still use the
> exec form to invoke executables.
- CMD:
```
CMD ["executable", "params", "more params"]
CMD ["param1","param2"]
CMD command param1 param2 ...:
```
> The `CMD` sets default commands for the built image; it doesn't run the command
> during build time.
>
> If arguments are supplied at Docker run, the CMD configurations here
> are overridden. The syntax rule of CMD is almost identical to
> `RUN`;
> the first form is the exec form, and the third one is the shell form, which
> is the prepend a `/bin/sh -c` as well. There is another directive in which
> `ENTRYPOINT` interacts with `CMD`; three forms of `CMD` actually would be a
> prepend with `ENTRYPOINT` when a container starts. There can be many `CMD` directives in a
> Dockerfile, but only the last one will take effect.
- ENTRYPOINT:
```
ENTRYPOINT ["executable", "param1", "param2"]
ENTRYPOINT command param1 param2
```
> These two forms are, respectively, the exec form and the shell form, and the
> syntax rules are the same as `RUN`. The entry point is the default executable
> for an image. That is to say, when a container spins up,
> it runs the executable configured by the `ENTRYPOINT`. When the `ENTRYPOINT` is
> combined with `CMD` and docker run arguments, writing in a different form would
> lead to very diverse behavior.
>
> Here are the organized rules of their combinations:
> If the `ENTRYPOINT` is in shell form, then the `CMD` and Docker run
> arguments would be ignored. The command will become:
>
> `/bin/sh -c entry_cmd entry_params ...`
>
> If the `ENTRYPOINT` is in exec form and the Docker run arguments are specified,
> then the `CMD` commands are overridden. The runtime command would be:
>
> `entry_cmd entry_params run_arguments`
>
> If the `ENTRYPOINT` is in exec form and only `CMD` is configured, the
> runtime command would become the following for the three
> forms:
>
> `entry_cmd entry_parms CMD_exec CMD_parms`
>
> `entry_cmd entry_parms CMD_parms`
>
> `entry_cmd entry_parms /bin/sh -c CMD_cmd CMD_parms`
- ENV:
```
ENV key value
ENV key1=value1 key2=value2 ...
```
> The `ENV` instruction sets environment variables for the consequent instructions
> and the built image. The first form sets the key to the string after the first space,
> including special characters. The second form allows us to set multiple variables
> in a line, separated with spaces.
>
> If there are spaces in a value, either enclose it with
> double quotes or escape the space character. Moreover, the key defined with
> `ENV` also takes effect on variables in the same documents. See the following examples
> to observe the behavior of `ENV`:
```
FROM alpine
ENV key wD # aw
ENV k2=v2 k3=v\ 3 \
    k4="v 4"
ENV k_${k2}=$k3 k5=\"K\=da\"
RUN echo key=$key ;\
   echo k2=$k2 k3=$k3 k4=$k4 ;\
   echo k_\${k2}=k_${k2}=$k3 k5=$k5
```
> And the output during the Docker build would be:
```
...
---> Running in 738709ef01ad
key=wD # aw
k2=v2 k3=v 3 k4=v 4
k_${k2}=k_v2=v 3 k5="K=da"
...
```
- LABEL:
```
LABEL key1=value1 key2=value2 ... The usage of
```
> `LABEL` resembles `ENV`, but a label is stored only in the metadata section
> of the images and is used by other host programs instead of programs in a container.
>
> It deprecates the maintainer instruction in the following form:
> `LABEL maintainer=johndoe@example.com`
> And we can filter objects with labels if a command has the -f(--filter) flag.
>
> For example, `docker images --filter label=maintainer=johndoe@example.com`
> queries out the images labeled with the preceding maintainer.
- EXPOSE:
```
EXPOSE <port> [<port> ...]
```
> This instruction is identical to the --expose
> flag at docker run/create , exposing ports at the container created by the
> resulting image.
- USER:
```
USER <name|uid>[:<group|gid>]
```
> The `USER` instruction switches the user to run the subsequent instructions.
> However, it cannot work properly if the user doesn't exist in the image.
> Otherwise, you have to run adduser before using the `USER` directive.
- WORKDIR:
```
WORKDIR <path>
```
> This instruction sets the working directory to a certain path.
> The path would be created automatically if the path doesn't exist.
>
> It works like cd in a Dockerfile , as it takes both relative and absolute
> paths and can be used multiple times.
>
> If an absolute path is followed by a relative path, the result would be
> relative to the previous path:
```
WORKDIR /usr
WORKDIR src
WORKDIR app
RUN pwd
---> Running in 73aff3ae46ac
/usr/src/app
---> 4a415e366388
```
> Also, environment variables set with ENV take effect on the path.
- COPY:
```
COPY <src-in-context> ... <dest-in-container>
COPY ["<src-in-context>",... "<dest-in-container>"]
```
> This directive copies the source to a file or a directory in the building container.
> The source could be files or directories, as could be the destination.
>
> The source must be within the context path, as only files under the context path will be sent to
> the Docker daemon. Additionally, `COPY` makes use of .dockerignore to filter
> files that would be copied into the building container. The second form is for a use
case where the path contains spaces.
- ADD
```
ADD <src > ... <dest >
ADD ["<src>",... "<dest >"]
```
> `ADD` is quite analogous to COPY  in terms of functionality:
> moving files into an image. More than copying files, <src> can also be URL or a compressed file.
>
> If <src> is a URL, ADD  will download it and copy it into the image
>
> If <src> is inferred as a compressed file, it will be extracted into <dest> path.
- VOLUME
```
VOLUME mount_point_1 mount_point_2
VOLUME ["mount point 1", "mount point 2"]
```
> The VOLUME instruction creates data volumes at the given mount points.
>
> Once it has been declared during build time, any change in the data volume at
> consequent directives would not persist. Besides, mounting host directories in a
> Dockerfile or docker build isn't doable because of portability issues: there's no guarantee
> that the specified path would exist in the host. The effect of both syntax forms is
> identical; they only differ in syntax parsing; The second form is a JSON array, so
> characters such as "\" should be escaped.
- ONBUILD
```
ONBUILD [Other directives]
```
> allows you to postpone some instructions to later builds in the derived image.
>
> For example, we may have the following two Dockerfiles:
```
      --- baseimg ---
      FROM alpine
      RUN apk add --no-update git make
      WORKDIR /usr/src/app
      ONBUILD COPY . /usr/src/app/
      ONBUILD RUN git submodule init && \
              git submodule update && \
              make
      --- appimg ---
      FROM baseimg
      EXPOSE 80
      CMD ["/usr/src/app/entry"]
```
> The instruction then would be evaluated in the following order on docker build:
```
$ docker build -t baseimg -f baseimg .
---
FROM alpine
RUN apk add --no-update git make
WORKDIR /usr/src/app
---
$ docker build -t appimg -f appimg .
---
COPY . /usr/src/app/
RUN git submodule init   && \
    git submodule update && \
    make
EXPOSE 80
CMD ["/usr/src/app/entry"]
```

## Docker in production

### Edit container env vars 

#### With Docker < v1.13:

Stop service docker.

Edit this file:

```
/var/lib/docker/containers/[container-id]/config.json
```

#### With Docker > v1.13 :

```
docker exec \<container-id\> -e ENV_VAR=value
```

## Monitoring docker

### All

https://github.com/firehol/netdata

### Host

#### Disk usage

```
docker system df
```

#### Docker daemon log 

* Ubuntu (old using upstart ) - ```/var/log/upstart/docker.log```
* Ubuntu (new using systemd ) - ```sudo journalctl -fu docker.service```
* Boot2Docker - ```/var/log/docker.log```
* Debian GNU/Linux - ```/var/log/daemon.log```
* CentOS - ```/var/log/daemon.log | grep docker```
* CoreOS - ```journalctl -u docker.service```
* Fedora - ```journalctl -u docker.service```
* Red Hat Enterprise Linux Server - ```/var/log/messages | grep docker```
* OpenSuSE - ```journalctl -u docker.service```
* OSX - ```~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/log/d‌​ocker.log```
* Windows - Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time

### Containers

#### Containers stats

/!\ It's fail in docker version < 17.12 if we have one container in creating state. Try without option --no-stream to solve it. 

```
docker stats --no-stream
```

#### Containers logs 

An example with a temporarily windowed logs selection (/!\ Be carefull --until is only available with Docker API 1.35+)

```
docker logs -t --since 2018-05-16T20:00:37 --until 2018-05-16T20:33:37 <container-id>
```

## Clean docker 

### General clean 

```
docker system prune
```

### Clean volumes 

```
 docker volume prune
 docker volume rm $(docker volume ls -qf dangling=true)
 docker volume ls -qf dangling=true | xargs -r docker volume rm
 ```

### Containers 

#### remove exited container.

```
docker rm $(docker ps –q –f status=exited)
```

#### remove exited container.

```
docker ps -a|grep Exit|cut -d' ' -f 1|xargs docker rm
```

### Images

#### remove unused images.

```
docker rmi –f $(docker images –q –f “dangling=true”)
```

#### remove images as possible that has no name/tag

```
docker images -a|grep '^<none>'|tr -s ' '|cut -d' ' -f 3|xargs docker rmi
```

## Purge docker

### Ubuntu 16.04

```
sudo apt-get purge -y docker-engine docker docker.io docker-ce  
sudo apt-get autoremove -y --purge docker-engine docker docker.io docker-ce 
sudo rm -rf /var/lib/docker
sudo rm /etc/apparmor.d/docker
sudo groupdel docker
sudo rm -rf /var/run/docker.sock
```

If you experiencing this issue https://github.com/moby/moby/issues/17902 (Unable to remove filesystem for xxx: remove /var/lib/docker/containers/xxx/shm: device or resource busy)

- Reboot the server
- Run again 

```
sudo rm -rf /var/lib/docker
```

## Other usefull commands

### Get file content in stdout 

```
docker exec <container-id> cat /filename
```
