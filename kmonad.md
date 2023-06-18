# Kmonad

KMonad is an advanced tool that lets you infinitely customize and extend the functionalities of almost any keyboard.

[github](https://github.com/kmonad/kmonad)

## Installation

```bash
curl -sSL https://get.haskellstack.org/ | sh # install stack

cd ~/src
git clone git@github.com:kmonad/kmonad.git
cd kmonad
stack build # build the project
stack haddock # generate documentation
stack install # install the project
```

## Permissions

[FAQ](https://github.com/kmonad/kmonad/blob/master/doc/faq.md#q-how-do-i-get-uinput-permissions)

```bash
# create a group for uinput
sudo groupadd uinput
# add your user to the uinput and input group
sudo usermod -aG uinput $USER
sudo usermod -aG input $USER
# create a udev rule to allow access to uinput
echo "KERNEL==\"uinput\", MODE=\"0660\", GROUP=\"uinput\", OPTIONS+=\"static_node=uinput\"" | sudo tee /lib/udev/rules.d/99-uinput.rules
# reload udev rules
sudo modprobe uinput
# reboot
```

## Kmonad configuration for Linux

```conf
# | --------------------------------------------------------------------------

  Kmonad configuration that does the following:
  
* when Caps is pressed and held:
  * i/k/j/l -> up/down/left/right
  * h -> backspace
  * w/s -> pageup/pagedown
* when lmet is pressed and held:
  * add a num pad to right side of the keyboard

  -------------------------------------------------------------------------- |#

(defcfg
  ;; For Linux
  input  (device-file "/dev/input/by-id/usb-04b4_4042-event-kbd")
  output (uinput-sink "caps-ikjl"
    ;; To understand the importance of the following line, see the section on
    ;; Compose-key sequences at the near-bottom of this file.
    "/run/current-system/sw/bin/sleep 1 && /run/current-system/sw/bin/setxkbmap -option compose:ralt")
  cmp-seq ralt    ;; Set the compose key to `RightAlt'
  cmp-seq-delay 5 ;; 5ms delay between each compose-key sequence press

  ;; For Windows
  ;; input  (low-level-hook)
  ;; output (send-event-sink)

  ;; For MacOS
  ;; input  (iokit-name "my-keyboard-product-string")
  ;; output (kext)

  ;; Comment this if you want unhandled events not to be emitted
  fallthrough true

  ;; Set this to false to disable any command-execution in KMonad
  allow-cmd false
)

(defsrc
  grv  1    2    3    4    5    6    7    8    9    0    -    =    bspc
  tab  q    w    e    r    t    y    u    i    o    p    [    ]    \
  caps a    s    d    f    g    h    j    k    l    ;    '    ret
  lsft z    x    c    v    b    n    m    ,    .    /    rsft
  lctl lmet lalt           spc            ralt rmet cmp  rctl
)

(deflayer qwerty
  grv  1    2    3    4    5    6    7    8    9    0    -    =    bspc
  tab  q    w    e    r    t    y    u    i    o    p    [    ]    \
  @cps a    s    d    f    g    h    j    k    l    ;    '    ret
  lsft z    x    c    v    b    n    m    ,    .    /    rsft
  lctl @num lalt           spc            ralt rmet cmp  rctl
)

(deflayer caps-ikjl
  _    _    _    _    _    _    _    _    _    _    _    _    _    _
  __    pgup __    __    _up_    __    __
  XX   _pgdn_    __    bspc left down rght __    _
  _    _    _    _    _    _    _    _    _    _    _    _
  _    _    _              _              _    _    _    _
)

(deflayer numbers
  _    _    _    _    _    _    _    _    _    _    _    _    _    _
  __    __    _XX   /    7    8    9    -_    __
  __    __    _XX   *    4    5    6    +_    _
  _    _\(   \)   .    XX   0    1    2    3_    _
  _    _    _              _              _    _    _    _
)

(defalias
  cps (layer-toggle caps-ikjl)
  num (tap-hold-next 400 lmet (layer-toggle numbers) :timeout-button lmet)
  stc (tap-hold-next 400 lsft caps :timeout-button lsft)
)
```

## Convinience scripts

```bash
#!/bin/bash

log='~/.kmonad/kmonad-daemon-keychron-k2.log'

now=$(date +"%Y-%m-%d %T")
echo "Kmonad activation keychron-k2\nExecution time: $now" >> $log

# keychron Keys
kbd='/dev/input/by-id/usb-USB_Keychron_K2_USB_DEVICE-event-kbd'
echo "Keychron K2 input event: "$kbd >> $log
file_path='/tmp/keychron_k2_kmonad_config.kbd'
if [ -e $kbd ]; 
then
    ~/anaconda3/bin/python \
        ~/.kmonad/write_kmonad_config.py \
        $file_path $kbd >> $log 2>&1
    nohup kmonad $file_path >> $log 2>&1 &
    echo "process id: $!"
    echo "Kmonad for Keychron K2 activated"
    echo "process id: $!" >> $log
    echo "Kmonad for Keychron K2 activated" >> $log
else
    echo "Keychron K2 not found; skipped"
    echo "Keychron K2 not found; skipped" >> $log
fi
```

## Resources

* [kmonad for bluetooth keyboard](https://github.com/MaxGyver83/dotfiles/blob/master/scripts/bin/start-kmonad-for-all-keyboards.fish)
* <https://github.com/kmonad/kmonad/issues/384>
* <https://github.com/kmonad/kmonad/blob/master/doc/quick-reference.md>
