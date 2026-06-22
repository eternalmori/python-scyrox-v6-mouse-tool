# python-scyrox-v6-mouse-tool
! This might break the support for Pulsar mice, I am not sure, since I only have a Scyrox v6 to test on, and I'm using it to simply read battery so I can pipe it into my taskbar, not much else.

Fork of [python-pulsar-mouse-tool](https://github.com/andrewrabert/python-pulsar-mouse-tool) with extended hardware support.

A tool to view/edit the on-device settings of Pulsar/Scyrox mice (same hardware family, vendor ID `0x3554`). 
An alternative to the Windows-only Pulsar Fusion software.

Supported mice:
- Pulsar X2 v2 Mini (`0xf507` wired, `0xf508` wireless dongle)
- Scyrox V6/V8 (`0xf5f6` wired, `0xf5f7` 8K wireless dongle)

Features:
- view and modify settings stored on the mouse hardware
- view battery information
- up to 8 DPI modes supported

## History
The protocol was reverse engineered by [Andrew Rabert](https://github.com/andrewrabert) using Wireshark to inspect USB packets
when the mouse was passed-through to a Windows virtual machine running the official Pulsar Fusion software.

This fork adds support for the Scyrox V6/V8 (same vendor ID `0x3554`, different product IDs) along with:
- Extended DPI mode slots from 4 to 8
- A race condition fix in settings reading (discard unsolicited HID events during bulk reads)


## Setup
Requires Python 3.7+ and [pyusb](https://github.com/pyusb/pyusb).

```bash
python3 -m venv venv
venv/bin/pip install pyusb
```

**Note:** You'll need to add the [udev rule](49-pulsar-mouse.rules) to access the device without root:

```bash
sudo cp 49-pulsar-mouse.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules
# unplug and re-plug the mouse
```

Then run:
```
venv/bin/python pulsar.py
```

## Usage
```
$ ./pulsar.py --help
usage: pulsar.py [-h] [--dpi DPI] [--dpi-mode DPI_MODE] [--led-brightness LED_BRIGHTNESS] [--led-color LED_COLOR]
                 [--led-effect {off,steady,breathe}] [--motion-sync {on,off}] [--lod-ripple {on,off}] [--angle-snapping {on,off}]
                 [--polling-rate {1000,500,250,125}] [--restore]

options:
  -h, --help            show this help message and exit
  --dpi DPI
  --dpi-mode DPI_MODE
  --led-brightness LED_BRIGHTNESS
  --led-color LED_COLOR
  --led-effect {off,steady,breathe}
  --motion-sync {on,off}
  --lod-ripple {on,off}
  --angle-snapping {on,off}
  --polling-rate {1000,500,250,125}
  --restore             restore factory-default settings
```


## Examples
### Retreive Current Settings and Battery Status
```
$ ./pulsar.py 
{
  "active_dpi_mode": 0,
  "active_profile": 0,
  "angle_snapping_enabled": false,
  "autosleep_seconds": 60,
  "debounce_milliseconds": 3,
  "dpi_modes": [
    {
      "dpi": 400,
      "dpi_mode": 0,
      "led_color": "#2c2d2e"
    },
    {
      "dpi": 800,
      "dpi_mode": 1,
      "led_color": "#303132"
    },
    {
      "dpi": 1600,
      "dpi_mode": 2,
      "led_color": "#343536"
    },
    {
      "dpi": 3200,
      "dpi_mode": 3,
      "led_color": "#38393a"
    }
  ],
  "led": {
    "effect": null,
    "enabled": false
  },
  "lod": {
    "mm": 1,
    "ripple_enabled": false
  },
  "motion_sync_enabled": true,
  "polling_rate_hz": 1000,
  "power": {
    "battery_millivolts": 3871,
    "battery_percent": 50,
    "connected": false
  }
}
```

### Set the active DPI mode's DPI
```
$ ./pulsar.py --dpi 400
```

## Adding a new mouse

If your mouse has vendor ID `0x3554` (compx/Pulsar/Scyrox) but a different product ID, run `lsusb` to find it, then add it to the `Device` class in `pulsar.py`:

```python
YOUR_MOUSE_DEVICE_ID = 0x????  # from lsusb
```

Add it to the scan loop in `Device.__init__` and update `49-pulsar-mouse.rules` if needed (the included rule matches all devices from vendor `0x3554`).
