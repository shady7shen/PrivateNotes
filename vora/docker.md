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
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTYzNDk5NTYwM119
-->