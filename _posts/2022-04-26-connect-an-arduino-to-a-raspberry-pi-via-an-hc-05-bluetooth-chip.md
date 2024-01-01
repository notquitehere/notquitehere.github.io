---
title: "Connect an Arduino to a Rasberry Pi via an HC-05 Bluetooth Module"
date: 2022-04-26
modified: 2022-11-08
published: true
excerpt: "Walk through of how to connect an HC-05 bluetooth module to a Raspbery Pi and setting up a service to automatically connect the device on startup and disconnect."
cover:
---

The £5 [HC-05 Bluetooth Transcever Module](https://thepihut.com/products/hc-05-bluetooth-module) is a popular choice for Arduino and Raspberry Pi projects however, it can be a touch fickle when it comes to connecting especially in the more recent iterations of [Rasbperry Pi OS](https://www.raspberrypi.com/software/).

## Add the serial port profile to the pi

To do this you need to edit the `dbus-org.bluez.service` file. (Use `nano` instead of `vim` if you haven't used vim before.)

```shell
sudo vim /etc/systemd/system/dbus-org.bluez.service
```

In the `[service]` section you need to add the compatability flag (`-C`)at the end of the `ExecStart` line and then add `ExecStartPost=/usr/bin/sdptool add SP` below it:

```shell
ExecStart=/usr/libexec/bluetooth/bluetoothd -C
ExecStartPost=/usr/bin/sdptool add SP
```

Restart the services:

```shell
sudo systemctl daemon-reload
sudo systemctl restart dbus-org.bluez.service
```

## Pair and trust your device

The simplest way to do this is to run `bluetoothctl` in interactive mode.

```shell
bluetoothctl
```

Assuming you didn't have any syntax errors in your file, this will change the prompt to `[bluetooth]#`. Run:

```shell
scan on
```

You'll be presented with a list of available bluetooth devices one of which should be: `00:00:00:00:00:00 HC-05` make a note of the address. If you can't see your device you might need to put it into pairing mode. Now run:

```shell
pair <00:00:00:00:00:00>
```

If your device has a PIN set you will be prompted to enter it. Once you've paired you need to run

```shell
trust <00:00:00:00:00:00>
```

to get out of bluetoothctl type `exit`

## Test your connection

Run

```shell
sudo rfcomm connect hci0 <00:00:00:00:00:00> <channel>
```

This is telling the internal bluetooth module (`hci0`) to connect to your HC-05 module.  If you don't specify a channel it will use 1. This should give you something like

```shell
connected /dev/rfcomm0 to 00:00:00:00:00:00 on channel 1
Press CTRL-C for hangup
```

If your bluetooth device is sending data you can trivially check this by running `cat /dev/rfcomm0` in a different terminal window.

NOTE: if you try to connect using `bluetoothctl` you will get `Failed to connect: org.bluez.Error.NotAvailable`.

## Connect on boot

The neatest way to do this is to create a service.

```shell
sudo vim /etc/systemd/system/<connectbt>.service
```

Add the following:

```shell
[Unit]
Description=Automatic connection to a bluetooth device
After=bluetooth.target
Requires=bluetooth.target

[Service]
User=root
ExecStart=rfcomm connect hci0 <00:00:00:00:00:00>
Restart=always
RestartSec=20

[Install]
WantedBy=multi-user.target
```

Reducing the restart interval from the default 100ms gives time for the user to turn the bluetooth device on in case they forget. If you don't do this you will hit the default `StartLimitBurst` (5) and the service will terminate.

If you want to check for the device more frequently it would be prudent to set `StartLimitIntervalSec` and `StartLimitBurst` in the `[Unit]` section. Be careful to ensure that `StartLimitBurst` * `RestartSec` < `StartLimitIntervalSec`.

Start your service:

```shell
sudo systemctl daemon-reload
sudo systemctl start <connectbt>.service
```

### Troubleshooting your script

If your service fails you can find the error message by running

```shell
journalctl -e -u <connectbt>
```

- `-e` drops you in at the end of the pager output
- `-u` limits the logs to those from the specified unit
- if you don't want the whole output you can also add `-n <lines>` to limit it to the last n lines
- If you want to check whether the service is connecting properly on failure you can add `-f` to tail the log. `CTRL-C` to exit.


## Resources:

- Getting the Pi to connect to the module: <a href="https://forums.raspberrypi.com/viewtopic.php?p=947185#p947185">https://forums.raspberrypi.com/viewtopic.php?p=947185#p947185</a>
- rfcomm: <a href="https://www.tutorialspoint.com/unix_commands/rfcomm.htm">https://www.tutorialspoint.com/unix_commands/rfcomm.htm</a>
- Managing Bluetooth Devices on Linux Using bluetoothctl: <a href="https://www.makeuseof.com/manage-bluetooth-linux-with-bluetoothctl/">https://www.makeuseof.com/manage-bluetooth-linux-with-bluetoothctl/</a>
- Serial Bluetooth Communication: <a href="https://pirobotblog.wordpress.com/2016/12/22/serial-bluetooth-communication/">https://pirobotblog.wordpress.com/2016/12/22/serial-bluetooth-communication/</a>
- How to Use journalctl to read Linux System Logs: <a href="https://www.howtogeek.com/499623/how-to-use-journalctl-to-read-linux-system-logs/">https://www.howtogeek.com/499623/how-to-use-journalctl-to-read-linux-system-logs/</a>
- Systemd Service Options: <a href="https://www.freedesktop.org/software/systemd/man/systemd.service.html#Options">https://www.freedesktop.org/software/systemd/man/systemd.service.html#Options</a>
- Systemd Unit Options: <a href="https://www.freedesktop.org/software/systemd/man/systemd.unit.html#%5BUnit%5D%20Section%20Options">https://www.freedesktop.org/software/systemd/man/systemd.unit.html#[Unit] Section Options</a>


