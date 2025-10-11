# Documentation for reproduction

How to setup chatbot runtime environment

## Setup

1. Setup Ubuntu Server with Docker on x86_64 machine (others untested)
    - `snap install docker && snap refresh --hold docker`
    - [`modprobe binder_linux devices="binder,hwbinder,vndbinder"`](https://github.com/remote-android/redroid-doc)
    - Start docker compose stack with [`docker-compose.yml` below](#docker-composeyml)
2. Make SSH tunnel (`ssh -n -L 55550:localhost:5555 192.168.0.42`)
3. `adb connect localhost:55550`
4. Download KakaoTalk APK from APKMirror or similar, and unzip
5. `adb -s localhost:55550 install-multiple APK_DIR/*.apk`
6. `scrcpy -s localhost:55550` and setup KakaoTalk account
7. Download `Iris.apk` from [`Iris` release page](https://github.com/dolidolih/Iris/releases)
8. `adb -s localhost:55550 push Iris.apk /data/local/tmp/Iris.apk`
9. Make [`/data/mjy/start.sh` below](#startsh) and `/data/mjy/logs/`:
    - `adb -s localhost:55550 shell`
        - `su 0 sh`
            - `mkdir -p /data/mjy/logs && echo 'content' > dest`
10. (optional) Teardown: `adb disconnect` and stop SSH tunnel

### `docker-compose.yml`

```yml
services:
  redroid:
    image: redroid/redroid:11.0.0-latest
    container_name: redroid
    privileged: true
    tty: true
    stdin_open: true
    ports:
      - "127.0.0.1:5555:5555"
      - "127.0.0.1:3000:3000"
      - "172.17.0.1:3000:3000"
    volumes:
      - ./data:/data
    restart: unless-stopped
    extra_hosts:
      - "host.docker.internal:host-gateway"
    command:
      - ro.dalvik.vm.native.bridge=libndk_translation.so
```

### `start.sh`

```sh
#!/bin/sh

set -e

CLASSPATH=/data/local/tmp/Iris.apk nohup app_process / party.qwer.iris.Main >> /data/mjy/logs/stdout.log 2>> /data/mjy/logs/stderr.log &
```

## Run

1. Make sure redroid container is running on server:
    - [`modprobe binder_linux devices="binder,hwbinder,vndbinder"`](https://github.com/remote-android/redroid-doc)
    - Up docker stack with the `docker-compose.yml` previously set
2. Run `/data/mjy/start.sh` on redroid container:
    - Make SSH tunnel if not yet
    - `adb connect localhost:55550` if not yet
    - Run `/data/mjy/start.sh`
        - `adb -s localhost:55550 shell`
            - `su 0 sh`
                - `sh /data/mjy/start.sh`
3. Run others if needed
    - For example, `adb -s localhost:5555 reverse tcp:5172 tcp:5172` on the server if webhook URL is set to localhost:5172
        - (Just an example — it's better to use `crontab` for this, as `adb reverse` may stop unexpectedly.)

### Iris Dashboard

You can access Iris dashboard after make proper SSH tunnel, like `ssh -L 30000:localhost:3000 192.168.0.42`

## Security Considerations

These servers don't need to have any ports exposed to the internet.

However, if you need to expose some ports for remote control purposes,
make sure that ADB or Iris ports (5555, 3000 respectively) are not exposed.  
It is recommended to expose only SSH with password authentication disabled (key-only) and other HTTPS ports.
