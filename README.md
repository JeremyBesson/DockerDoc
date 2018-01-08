# DockerDoc

## Clean docker 

### remove exited container.

```
docker rm $(docker ps –q –f status=exited)
```

### remove unused images.

docker rmi –f $(docker images –q –f “dangling=true”)

### remove exited container.

```
docker ps -a|grep Exit|cut -d' ' -f 1|xargs docker rm
```

### remove images as possible that has no name/tag

```
docker images -a|grep '^<none>'|tr -s ' '|cut -d' ' -f 3|xargs docker rmi
```
