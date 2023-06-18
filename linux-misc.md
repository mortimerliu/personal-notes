# Linux

## Ubuntu 22.04 Post-Installation

- ensential

    ```bash
    sudo apt install nvidia-driver-530
    sudo apt install git
    sudo apt-get install ibus-libpinyin # for Chinese input, reboot needed
    ```

- git-config, see [here](git.md)
- Java

    ```bash
    sudo apt install openjdk-17-jre
    sudo apt install openjdk-17-jdk

    # set JAVA_HOME
    sudo echo "JAVA_HOME=\"/usr/lib/jvm/java-11-openjdk-amd64\"" >> /etc/environment
    source /etc/environment
    echo $JAVA_HOME

    # configure java alternatives
    sudo update-alternatives --config java
    sudo update-alternatives --config javac
    ```

- logid, see [here](logid.md)
- docker, see [here](docker.md)
- kmonad, see [here](kmonad.md)

## Commands

create symlink

```bash
ln -s <src> <dest>
```

date

```bash
date +"%Y-%m-%d %T"
```

store command output to a variable

```bash
var=$(cmd)
```

- `$!` stores the process id of last command
- `cd -` return to the previous directory (not the parent!)

## Display not waking up after suspend model

- (not working)[StackExchange](https://askubuntu.com/questions/1032633/18-04-screen-remains-blank-after-wake-up-from-suspend)
- (working) Use NVIDIA driver instead of the Nouveau display driver

## Resources

- [bash-escape-characters](https://www.baeldung.com/linux/bash-escape-characters)
- [check file existence](https://stackoverflow.com/questions/17420994/how-can-i-match-a-string-with-a-regex-in-bash)
