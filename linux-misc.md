# Linux

## Commands

create symlink

```bash
ln -s <src> <dest>
```

date

```
date +"%Y-%m-%d %T"
```

store command output to a variable

```
var=$(cmd)
```

* `$!` stores the process id of last command
* `cd -` return to the previous directory (not the parent!)

## Display not waking up after suspend model

* (not working)[StackExchange](https://askubuntu.com/questions/1032633/18-04-screen-remains-blank-after-wake-up-from-suspend)
* (working) Use NVIDIA driver instead of the Nouveau display driver

## Resources

* [bash-escape-characters](https://www.baeldung.com/linux/bash-escape-characters)
* [check file existence](https://stackoverflow.com/questions/17420994/how-can-i-match-a-string-with-a-regex-in-bash)