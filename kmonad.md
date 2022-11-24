# Kmonad

## Kmonad configuration

```
#| --------------------------------------------------------------------------

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
  _    _    pgup _    _    _    _    _    up   _    _    _    _    _
  XX   _    pgdn _    _    _    bspc left down rght _    _    _
  _    _    _    _    _    _    _    _    _    _    _    _
  _    _    _              _              _    _    _    _
)

(deflayer numbers
  _    _    _    _    _    _    _    _    _    _    _    _    _    _
  _    _    _    _    _    XX   /    7    8    9    -    _    _    _
  _    _    _    _    _    XX   *    4    5    6    +    _    _
  _    _    \(   \)   .    XX   0    1    2    3    _    _
  _    _    _              _              _    _    _    _
)

(defalias
  cps (layer-toggle caps-ikjl)
  num (tap-hold-next 400 lmet (layer-toggle numbers) :timeout-button lmet)
  stc (tap-hold-next 400 lsft caps :timeout-button lsft)
)
```