# `docker-OSX`: Installl macOS on Linux

https://dev.to/gombosg/running-macos-inside-linux-with-docker-osx-4e1i#cpu-virtualization

## Big Sur

```bash
docker run -it \
    --device /dev/kvm \
    -e AUDIO_DRIVER=pa,server=unix:/tmp/pulseaudio.socket \
    -p 50922:10022 \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -e "DISPLAY=${DISPLAY:-:0.0}" \
    -e PULSE_SERVER=unix:/tmp/pulseaudio.socket \
    -v "${SHARE}:/mnt/hostshare" \
    -e EXTRA='-smp 16,sockets=8,cores=2' \
    sickcodes/docker-osx:big-sur
    
# docker build -t docker-osx --build-arg SHORTNAME=big-sur .
```

docker run -it \
    --device /dev/kvm \
    -p 50922:10022 \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -e "DISPLAY=${DISPLAY:-:0.0}" \
    -v "${PWD}/mac_hdd_ng.img:/home/arch/OSX-KVM/mac_hdd_ng.img" \
    -v "${SHARE}:/mnt/hostshare" \
    -e EXTRA="-virtfs local,path=/mnt/hostshare,mount_tag=hostshare,security_model=passthrough,id=hostshare" \
    sickcodes/docker-osx:latest

# !!! Open Terminal inside macOS and run the following command to mount the virtual file system
# sudo -S mount_9p hostshare

### Use more CPU Cores/SMP

```bash
-e EXTRA='-smp 6,sockets=3,cores=2'
-e EXTRA='-smp 8,sockets=4,cores=2'
-e EXTRA='-smp 16,sockets=8,cores=2'
```

Note, unlike memory, CPU usage is shared. so you can allocate all of your CPU's to the container.

### Share directories/files

```bash
# on Linux/Windows
mkdir ~/mnt/osx
sshfs user@localhost:/ -p 50922 ~/mnt/osx
# wait a few seconds, and ~/mnt/osx will have full rootfs mounted over ssh, and in userspace
# automated: sshpass -p <password> sshfs user@localhost:/ -p 50922 ~/mnt/osx
```

### iPhone USB passthrough

* Option 1: https://github.com/Silfalion/Iphone_docker_osx_passthrough

## pass audio not working
* https://stackoverflow.com/questions/59988019/emulator-pulseaudio-access-denied
* https://github.com/TheBiggerGuy/docker-pulseaudio-example/issues/2


## OSX-Optimizer

https://github.com/sickcodes/osx-optimizer


## Reference
* [Blog](https://www.linuxuprising.com/2021/03/install-macos-big-sur-or-catalina-in.html)
* [qemu](https://www.qemu.org/)
* [github repo](https://github.com/sickcodes/Docker-OSX)



```bash
# base image to use
IMAGE=sickcodes/docker-osx:big-sur
# folder to share with the macOS vm
SHARE=/home/hongruliu


docker run -it \
    --device /dev/kvm \
    -e AUDIO_DRIVER=pa,server=unix:/tmp/pulseaudio.socket \
    -p 50922:10022 \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -e "DISPLAY=${DISPLAY:-:0.0}" \
    -e PULSE_SERVER=unix:/tmp/pulseaudio.socket \
    -v "${SHARE}:/mnt/hostshare" \
    -e EXTRA='-smp 16,sockets=1,cores=8,threads=2 -m 8G' \
    ${IMAGE}


docker run -it \
    --device /dev/kvm \
    -e AUDIO_DRIVER=pa,server=unix:/tmp/pulse-socket \
    -p 50922:10022 \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -e "DISPLAY=${DISPLAY:-:0.0}" \
    -e PULSE_SERVER=unix:/tmp/pulseaudio.socket \
    -v "/home/hongruliu:/mnt/hostshare" \
    -e EXTRA='-smp 16,sockets=1,cores=8,threads=2 -m 8G' \
sickcodes/docker-osx:big-sur


docker run -it \
    --device /dev/kvm \
    --network host \
    -v "/home/hongruliu/.docker-osx/mac_hdd_ng.img:/image" \
    -v "/home/hongruliu/.docker-osx/shared:/mnt/hostshare" \
    -e AUDIO_DRIVER=pa,server=unix:/tmp/pulse-socket \
    -p 50922:10022 \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -e "DISPLAY=${DISPLAY:-:0.0}" \
    -e PULSE_SERVER=unix:/tmp/pulseaudio.socket \
    -e EXTRA='-smp 16,sockets=1,cores=8,threads=2 -m 16G' \
    sickcodes/docker-osx:naked


docker run \
    --device /dev/kvm \
    --network host \
    -v "$HOME/mac/mac_hdd_ng.img:/image" \
    -v "$HOME/mac/shared:/mnt/hostshare" \
    -e EXTRA="-virtfs local,path=/mnt/hostshare,mount_tag=hostshare,security_model=passthrough,id=hostshare" \
    -e RAM=8 \
    -e CPU_STRING=16 \
    docker-osx:nakedvnc


## vnc password: 9VdLz5~=


docker run -it \
    --device /dev/kvm \
    --network host \
    -v "/home/hongruliu/.docker-osx/mac_hdd_ng.img:/image" \
    -v "/home/hongruliu/.docker-osx/shared:/mnt/hostshare" \
    -e AUDIO_DRIVER=pa,server=unix:/tmp/pulse-socket \
    -e PULSE_SERVER=unix:/tmp/pulseaudio.socket \
    -e EXTRA='-smp 16,sockets=1,cores=8,threads=2 -m 16G' \
    docker-osx:nakedvnc
```