# MicroPython Winbond Flash

[![Downloads](https://pepy.tech/badge/micropython-winbond)](https://pepy.tech/project/micropython-winbond)
![Release](https://img.shields.io/github/v/release/brainelectronics/micropython-winbond?include_prereleases&color=success)
![MicroPython](https://img.shields.io/badge/micropython-Ok-green.svg)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![CI](https://github.com/brainelectronics/micropython-winbond/actions/workflows/release.yml/badge.svg)](https://github.com/brainelectronics/micropython-winbond/actions/workflows/release.yml)

MicroPython library to interact with Winbond W25Q Flash chips

-----------------------

<!-- MarkdownTOC -->

- [Installation](#installation)
	- [Install required tools](#install-required-tools)
- [Stetup](#stetup)
	- [Install package with pip](#install-package-with-pip)
	- [Manually](#manually)
		- [Upload files to board](#upload-files-to-board)
	- [Open REPL \(in rshell\)](#open-repl-in-rshell)
- [Credits](#credits)

<!-- /MarkdownTOC -->

## Installation

### Install required tools

Python3 must be installed on your system. Check the current Python version
with the following command

```bash
python --version
python3 --version
```

Depending on which command `Python 3.x.y` (with x.y as some numbers) is
returned, use that command to proceed.

```bash
python3 -m venv .venv
source .venv/bin/activate

pip install -r requirements.txt
```

For interaction with the filesystem of the device the
[Remote MicroPython shell][ref-remote-upy-shell] can be used.

Test the tool by showing its man/help info description.

```bash
rshell --help
```

## Stetup

### Install package with pip

Connect your MicroPython board to a network

```python
import network
station = network.WLAN(network.STA_IF)
station.connect('SSID', 'PASSWORD')
station.isconnected()
```

and install this lib on the MicroPython device like this

```python
import upip
upip.install('micropython-winbond')
```

### Manually

#### Upload files to board

Copy the module to the MicroPython board and import them as shown below
using [Remote MicroPython shell][ref-remote-upy-shell]

Open the remote shell with the following command. Additionally use `-b 115200`
in case no CP210x is used but a CH34x.

```bash
rshell -p /dev/tty.SLAB_USBtoUART --editor nano
```

Check the board config with this simple `boards` call inside the rshell. The
result will look similar to this after the connection

```bash
Using buffer-size of 32
Connecting to /dev/tty.SLAB_USBtoUART (buffer-size 32)...
Trying to connect to REPL  connected
Retrieving sysname ... esp32
Testing if ubinascii.unhexlify exists ... Y
Retrieving root directories ... /boot.py/
Setting time ... Feb 17, 2022 08:56:14
Evaluating board_name ... pyboard
Retrieving time epoch ... Jan 01, 2000
Welcome to rshell. Use Control-D (or the exit command) to exit rshell.
/Users/Jones/Downloads/MicroPython/micropython-winbond> boards
pyboard @ /dev/tty.SLAB_USBtoUART connected Epoch: 2000 Dirs: /pyboard/boot.py
```

Perform the following command to copy all files and folders to the device

```bash
cp SOURCE_FILE_NAME /pyboard

# optional copy it as another file name
cp SOURCE_FILE_NAME /pyboard/NEW_FILE_NAME
```

Use these commands to download the files of this repo to the board.

```bash
cp winbond.py /pyboard
cp main.py /pyboard
cp boot.py /pyboard
```

### Open REPL (in rshell)

Call `repl` in the rshell. Use CTRL+X to leave the repl or CTRL+D for a soft
reboot of the device.

The [`boot.py`](boot.py) code will perform the following steps:

 * create a flash object on `SPI2` with a SPI speed of 2kHz, chip select on
   machine `pin 5` and a software reset interface. Check the datasheet of your
   flash chip whether software reset is supported.
 * try to mount the external flash to `/external`
 	 * in case of `OSError 19` the filesystem will be created first
 	 * otherwise the complete flash will be erased and the filesystem created
 	 * finally mount the external flash

The initial steps of formatting the flash and creating the filesystem will
take around 45 seconds on an ESP32 board like the [BE32-01][ref-be32] equipped
with a 128MBit (16MB) external Winbond flash.

The [`main.py`](main.py) code will perform the following steps:

 * list all files and folders on the boards root directory
 * try to read a file named `some-file.txt` on the external flash
 	 * if the file does not exist, it will be created
 * extend the content of `some-file.txt` on the external flash with new content
 * list all files and folders on the external flash directory

A successful output of a first run after a soft reboot should look similar to
this

```
MPY: soft reboot
manufacturer: 0xef
mem_type: 64
device: 0x4018
capacity: 16777216 bytes
Mounting the external flash to "/external" ...
Failed to mount the external flash due to: [Errno 19] ENODEV
Creating filesystem for external flash ...
This might take up to 10 seconds
Filesystem for external flash created
External flash mounted to "/external"
boot.py steps completed
Entered main.py
Listing all files and folders on the boards root directory "/":
['external', 'boot.py', 'main.py', 'winbond.py']
No test file "some-file.txt" exists on external flash "/external", creating it now
Listing all files and folders on the external flash directory "/external":
['some-file.txt']
Finished main.py code. Returning to REPL now
MicroPython v1.18 on 2022-01-17; ESP32 module with ESP32
Type "help()" for more information.
>>>
MPY: soft reboot
manufacturer: 0xef
mem_type: 64
device: 0x4018
capacity: 16777216 bytes
Mounting the external flash to "/external" ...
External flash mounted to "/external"
boot.py steps completed
Entered main.py
Listing all files and folders on the boards root directory "/":
['external', 'boot.py', 'main.py', 'winbond.py']
Test file "some-file.txt" exists on external flash "/external"
This is its content:
Hello World at 698411330

Appending new content to "/external/some-file.txt"
Listing all files and folders on the external flash directory "/external":
['some-file.txt']
Finished main.py code. Returning to REPL now
MicroPython v1.18 on 2022-01-17; ESP32 module with ESP32
Type "help()" for more information.
```

## Credits

Kudos and big thank you to [crizeo of the MicroPython Forum][ref-crizeo] and
his [post to use Winbond flash chips][ref-upy-forum-winbond-driver]

<!-- Links -->
[ref-remote-upy-shell]: https://github.com/dhylands/rshell
[ref-be32]: https://github.com/brainelectronics/BE32-01/
[ref-pfalcon-picoweb-sdist-upip]: https://github.com/pfalcon/picoweb/blob/b74428ebdde97ed1795338c13a3bdf05d71366a0/sdist_upip.py
[ref-test-pypi]: https://test.pypi.org/
[ref-pypi]: https://pypi.org/
[ref-invalid-auth-test-pypi]: https://test.pypi.org/help/#invalid-auth
[ref-crizeo]: https://forum.micropython.org/memberlist.php?mode=viewprofile&u=3067
[ref-upy-forum-winbond-driver]: https://forum.micropython.org/viewtopic.php?f=16&t=3899&start=10
