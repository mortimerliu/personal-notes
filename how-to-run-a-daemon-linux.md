# How to Run a Daemon in Linux

## `systemd`

some useful commands:

```
systemctl daemon-reload
systemctl [start|status|stop|enable|disable|reload] <some-daemon>
systemctl list-units
```

## Start a Daemon

Using `systemd` you should be able to run a script as a `daemon` by creating a simple `unit`.

start with a script that you want to run as `daemon` called `/usr/bin/mydaemon`.

```bash
#!/bin/sh

while true; do
  date;
  sleep 60;
done
```

Make sure the file is executable: 

```
sudo chmod +x /usr/bin/mydaemon
```

You create a unit `/etc/systemd/system/mydaemon.service`.

```
[Unit]
Description=My daemon
SourcePath=/usr/bin/mydaemon

[Service]
Type=forking
Restart=always
RestartSec=3s
TimeoutSec=5min
IgnoreSIGPIPE=no
KillMode=process
GuessMainPID=no
RemainAfterExit=yes
ExecStart=/usr/bin/mydaemon start
ExecStop=/usr/bin/mydaemon stop

[Install]
WantedBy=multi-user.target 
```

To start the demon:

```bash
systemctl start mydaemon.service 
```

To start at boot:

```
systemctl enable mydaemon.service
```

## References

* [proper-way-to-run-shell-script-as-a-daemon](https://unix.stackexchange.com/questions/426862/proper-way-to-run-shell-script-as-a-daemon)
* [how-to-use-systemctl](https://www.digitalocean.com/community/tutorials/how-to-use-systemctl-to-manage-systemd-services-and-units)
* [auto-restart-crashed-service-systemd](https://singlebrook.com/2017/10/23/auto-restart-crashed-service-systemd/)
* [init.d](https://superuser.com/questions/449805/init-d-script-not-running-on-startup)