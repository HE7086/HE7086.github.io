# udev

## remap keyboard using hwdb

### Create a configuration at `/etc/udev/hwdb.d/xxx.hwdb`:

* integrated keyboard of a laptop
```
evdev:atkbd:dmi:*
 KEYBOARD_KEY_01=capslock   # esc
 KEYBOARD_KEY_3a=esc        # capslock
```

* generic usb keyboard
  - `v....` : vendor id
  - `p....` : product id
  - `KEY_...` : hex scancode from `evtest`
```
evdev:input:b*v....p....e*-*
 KEYBOARD_KEY_...=capslock   # esc
 KEYBOARD_KEY_...=esc        # capslock
```

### reload config
* `systemd-hwdb update`
* `udevadm trigger`
