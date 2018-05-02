# DockerDoc

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

### Docker daemon log 

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

### Containers stats

/!\ It's fail in docker version < 17.12 if we have one container in creating state. Try without option --no-stream to solve it. 

```
docker stats --no-stream
```

## Clean docker 

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

## Purge docker (Unbuntu 16.04)

```
sudo apt-get purge -y docker-engine docker docker.io docker-ce  
sudo apt-get autoremove -y --purge docker-engine docker docker.io docker-ce  
sudo rm -rf /var/lib/docker
sudo rm /etc/apparmor.d/docker
sudo groupdel docker
sudo rm -rf /var/run/docker.sock
```

## Other usefull commands

### Get file content in stdout 

```
docker exec <container-id> cat /filename
```
