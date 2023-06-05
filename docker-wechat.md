# docker-wechat 

This is how to use WeChat on Ubuntu.

- For both methods, `docker-desktop` not working. Install `docker` throught `apt repository` as instructed [here](https://docs.docker.com/engine/install/ubuntu/)
- Run `xhost +` before running docker, and `xhost -` after.
- Ubuntu 22.04

## `deepin` based

- [Dockerfile](https://github.com/top-bettercode/docker-wechat)
- [Docker image](https://hub.docker.com/r/bestwu/wechat/)
- [Instructions](https://github.com/top-bettercode/docker-wechat#readme)
- https://github.com/Alice-space/docker-wechat
- If using `ibus` input method, change `fcitx` to `ibus` for all three environment variables `QT_IM_MODULE`, `XMODIFIERS=@im` and `GTK_IM_MODULE`

## Linux Wechat based

- [Dockerfile](https://github.com/leimao/Docker-WeChat)
- Cannot use Chinese input. Seems `ibus` is working but not sure. need to take a try


## Docker Basics

Example to run a minimal clock on Docker

```bash
cat > Dockerfile << EOF
FROM ubuntu:22.04
RUN apt-get update
RUN apt-get install -y x11-apps
CMD xclock
EOF

docker run -e DISPLAY \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    --user="$(id --user):$(id --group)" ubuntu:xclock
```