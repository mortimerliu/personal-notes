# `logid`

## `/etc/logid.cfg`


enable the service to run on system startup and start the service:

```
sudo systemctl enable logid
sudo systemctl start logid
```

If ever change /etc/logid.cfg, for the changes to take effect you can run:

```
sudo systemctl restart logid
```

```
# this config file is for Logiops and needs to be placed in /etc/logid.cfg
# name can be found by running `sudo logid`
devices: (
{
    name: "Wireless Mouse MX Master 2S";
    smartshift:
    {
        on: false;
        threshold: 15; # 7 is ideal for work
    };
    hiresscroll:
    {
        hires: false;
        invert: false;
        target: false;
    };
    dpi: 800;# <- you may change this number

    buttons: (
        {
            cid: 0xc3;
            action =
            {
                type: "Gestures";
                gestures: (
                    {
                        direction: "Up";
                        mode: "OnRelease";
                        action =
                        {
                            type: "Keypress";
                            keys: ["KEY_LEFTCTRL", "KEY_LEFTALT",  "KEY_UP"];
                        };
                    },
                    {
                        direction: "Down";
                        mode: "OnRelease";
                        action =
                        {
                            type: "Keypress";
                            keys: ["KEY_LEFTCTRL", "KEY_LEFTALT", "KEY_DOWN"];
                        };
                    },
                    {
                        direction: "Left";
                        mode: "OnRelease";
                        action =
                        {
                            type: "Keypress";
                            keys: ["KEY_LEFTCTRL", "KEY_LEFTALT", "KEY_LEFT"];
                        };
                    },
                    {
                        direction: "Right";
                        mode: "OnRelease";
                        action =
                        {
                            type: "Keypress";
                            keys: ["KEY_LEFTCTRL", "KEY_LEFTALT", "KEY_RIGHT"];
                        }
                    },

                    {
                        direction: "None"
                        mode: "OnRelease";
                        action =
                        {
                            type: "Keypress";
                            keys: ["KEY_LEFTMETA"];
                        }
                    }
                );
            };
        },
        {
            cid: 0xc4;
            action =
            {
                type = "ToggleSmartshift";
            };
        }
    );
}
);
```

## [scroll speed](https://github.com/PixlOne/logiops/issues/116)

```
hiresscroll:
{
    hires: true;
    invert: false;
    target: true;
    up: {
        mode: "Axis";
        axis: "REL_WHEEL_HI_RES";
        axis_multiplier: 1;
    },
    down: {
        mode: "Axis";
        axis: "REL_WHEEL_HI_RES";
        axis_multiplier: -1;
    },
}
```


## Resources

* [git repo](https://github.com/PixlOne/logiops/tree/master/src/logid)
* [how-to-use-logid](https://askubuntu.com/questions/1149310/logitech-mx-master-2s-via-bluetooth-change-pointer-speed/1246278#1246278)
* [configuration wiki](https://github.com/PixlOne/logiops/wiki/Configuration)