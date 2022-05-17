# Netopeer2 Docker setup

## netopeer2-server

[Source](https://github.com/sysrepo-archive/docker-sysrepo-netopeer2/blob/master/Dockerfile)

Remove all stopped containers, dangling images and unused networks:

```
docker system prune
```

```
docker build -t netopeer2-server .
docker run -i -t netopeer2-server
```

```
apt-get update
apt-get upgrade -y
apt-get autoclean
apt-get autoremove


# Source: https://serverfault.com/questions/949991/how-to-install-tzdata-on-a-ubuntu-docker-image
ARG_DEBIAN_FRONTEND=noninteractive
ENV_TZ=Europe/Tallinn
apt-get install -y tzdata

apt-get install git cmake build-essential bison flex libpcre3-dev libev-dev libavl-dev libprotobuf-c-dev protobuf-c-compiler swig python-dev lua5.2 pkg-config libpcre++-dev openssl libssl-dev libcrypto++-dev zlib1g-dev libpcre2-dev libssh-dev uncrustify libcmocka-dev

apt-get install -y wget

wget \
  https://git.libssh.org/projects/libssh.git/snapshot/libssh-0.9.6.tar.gz
tar -xf libssh-0.9.6.tar.gz
rm libssh-0.9.6.tar.gz
cd libssh-0.9.6
mkdir build
cd build
cmake ..
make
make install

cd ~
mkdir NetConfServer
cd NetConfServer

git clone https://github.com/CESNET/libyang.git
cd libyang
mkdir build
cd build
cmake ..
make && make install

cd ~/NetConfServer
git clone https://github.com/sysrepo/sysrepo.git
cd sysrepo
mkdir build
cd build
cmake ..
make && make install

cd ~/NetConfServer
git clone https://github.com/CESNET/libnetconf2.git
cd libnetconf2
mkdir build
cd build
cmake ..
make && make install

cd ~/NetConfServer
ldconfig /usr/local/lib
git clone https://github.com/CESNET/netopeer2.git
cd netopeer2
mkdir build
cd build
cmake ..
make && make install
```