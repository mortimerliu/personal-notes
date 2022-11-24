# `logid`

## `/etc/logid.cfg`

```
# this config file is for Logiops and needs to be placed in /etc/logid.cfg
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

## Resources

* [git repo]()
* [how-to-use-logid](https://askubuntu.com/questions/1149310/logitech-mx-master-2s-via-bluetooth-change-pointer-speed/1246278#1246278)