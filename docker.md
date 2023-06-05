# docker

## Change `docker` root directory

* docker by default stores all images and containers at `var/lib/docker`
* if you want to change this, potentially due to disk space limit, you can follow the steps below to change it to another directory
* [solution link](https://linuxconfig.org/how-to-move-docker-s-default-var-lib-docker-to-another-directory-on-ubuntu-debian-linux#:~:text=By%20default%2C%20Docker%20stores%20most,docker%20directory%20on%20Linux%20systems.)

### Step 1: stop docker daemon

```bash
sudo systemctl stop docker.service
sudo systemctl stop docker.socket
```

### Step 2: change service config

edit the `/lib/systemd/system/docker.service` file, change the line similar to 

```
ExecStart=/usr/bin/dockerd -H fd://
```

to

```
ExecStart=/usr/bin/dockerd -g /path/to/new/dir/docker -H fd://
```

also remember to create the new dir:

```bash
sudo mkdir -p /path/to/new/dir/docker
```

my choice: `/home/$USER/lib/docker`

### Step 3: copy

```bash
sudo rsync -aqxP /var/lib/docker/ /path/to/new/dir/docker
```

### Step 4: start docker daemon

```bash
sudo systemctl daemon-reload
sudo systemctl start docker
```

test out it's working:

```bash
docker run hello-world
```

also, check if the root dir is changed

```bash
docker info | grep Root
```

### Step 5 [Optional]: delete old dir

```bash
sudo rm -rf /var/lib/docker
```