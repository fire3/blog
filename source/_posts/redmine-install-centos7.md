title: redmine install centos7
date: 2016-12-20 08:54:56
tags: [docker, redmine]
---

first install docker and docker-compose:

follow the instruction here: 
https://docs.docker.com/engine/installation/linux/centos/
```
$ sudo tee /etc/yum.repos.d/docker.repo <<-'EOF'
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/7/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF

$ sudo yum install docker-engine
$ sudo systemctl enable docker.service
$ sudo systemctl start docker
$ sudo docker run --rm hello-world

$ sudo pip install docker-compose

```

then use docker install redmine

```
wget https://raw.githubusercontent.com/sameersbn/docker-redmine/master/docker-compose.yml
```
