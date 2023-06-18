# `logid`

[github](https://github.com/PixlOne/logiops)

## Installation

```bash
sudo apt install build-essential cmake pkg-config libevdev-dev libudev-dev libconfig++-dev libglib2.0-dev

cd ~/src
git clone git@github.com:PixlOne/logiops.git
cd logiops
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make
sudo make install
sudo cp ../logid.cfg /etc/logid.cfg # see below
sudo systemctl enable logid # enable the service to run on system startup
sudo systemctl start logid # start the service
```

## `/etc/logid.cfg`

- This config file is for Logiops and needs to be placed in `/etc/logid.cfg`
- Device name can be found by running `sudo logid`
- [scroll speed](https://github.com/PixlOne/logiops/issues/116)
- If ever change /etc/logid.cfg, for the changes to take effect you can run:

    ```bash
    sudo systemctl restart logid
    ```

```conf
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
        hires: true;
        invert: false;
        target: false;
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

- [how-to-use-logid](https://askubuntu.com/questions/1149310/logitech-mx-master-2s-via-bluetooth-change-pointer-speed/1246278#1246278)
- [configuration wiki](https://github.com/PixlOne/logiops/wiki/Configuration)
