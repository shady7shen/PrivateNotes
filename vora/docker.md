# Docker Installation
## In China (accelerated)
### AutoScript
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

### Manual Script
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"

### Adding China Docker Hub Mirror
shady@osboxes:/etc/docker$ cat daemon.json
{
  "registry-mirror": ["https://registry.docker-cn.com"]
}

### Allow User to Use Docker Commandline
sudo groupadd docker
sudo gpasswd -a $USER docker
newgrp docker

# Docker Commandline
## Working with Images
### List
`docker images -a --no-trunc [repository[:tag]]`
### Pull
```
docker pull [options] NAME[:tag|@digest]
--all-tags,-a download all tagged images in the repository
--disable-content-trust (default) skip image verification 
```
It's also possible to manually specify the path of a docker hub. For example, the following pulls image `/testing/test-image` from hub `myregistry.local:5000`
```
docker pull myregistry.local:5000/testing/test-image
```
### Import File System as an Image
```
docker import [options] file|URL|- [Repository[:tag]]

--change, -c apply Dockerfile instruction, supporting instructions: CMD | ENTRYPOINT | ENV | EXPOSE | ONBUILD | USER | VOLUME | WORKDIR
--message, -m set commit message for imported image

URL can point to an archive (.tar, .tar.gz, .tgz, .bzip, .tar.xz, or .txz) containing a file system or a single file on the Docker host
-   is instructing docker to read data from STDIN
```

### Save (multiple)
`docker save -o <tarFileName> IMAGE [IMAGE ...]` 
### Load (image)

## Working with Containers
### Export container file system to a tar




https://docs.docker.com/engine/reference/commandline/save/
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2MzIxNjExMjYsLTk4OTgwNjU5NV19
-->