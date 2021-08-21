#Docker
### List container
``` bash
docker ps -a
```
### Run container and remove on terminate
``` bash
docker run --rm -it container_name bin/bash
```
### Tag and push image
``` bash
docker tag myimage:1.0 myrepo/myimage:2.0
docker push myrepo/myimage:2.0
```
### List the running containers
``` bash
# add `--all` to include stopped containers
docker container ls
```
### Delete all running and stopped containers
``` bash
docker container rm -f $(docker ps -aq)
```
### Delete old containers
``` bash
docker ps -a | grep 'weeks ago' | awk '{print $1}' | xargs docker rm
```
### Delete stopped containers
``` bash
docker rm -v $(docker ps -a -q -f status=exited)
```
### Delete dangling images
``` bash
docker rmi $(docker images -q -f dangling=true)
```
### Delete all images
``` bash
docker rmi $(docker images -q)
```
### Print the last 100 lines of a container’s logs
``` bash
docker container logs --tail 100 CONTAINER_NAME
```
### Mapping the container port to the host port (only using localhost interface)
``` bash
docker run -p 127.0.0.1:$HOSTPORT:$CONTAINERPORT --name CONTAINER -t IMAGE
```
### Run cloc with docker
``` bash
docker run --rm -v $PWD:/tmp aldanial/cloc --vcs=git
```