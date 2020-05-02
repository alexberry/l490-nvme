# Lenovo Thinkpad L490 + Intel nvme SSD 760p

## Original issue

The original issue was posted on reddit by [/u/Kronikaetor](https://www.reddit.com/user/Kronikaetor/) and is available [here](https://www.reddit.com/r/linuxquestions/comments/g88hhs/need_help_installing_ubuntu/).

## Summary

Kronikaetor has an in-warranty Thinkpad L490. Initially shipped with [freedos](http://www.freedos.org/), he installed Ubuntu 19.04 until it reached EOL, then attempted to install 20.04 at which point his laptop refused to run stably. He attempted:

* Installation of 18.04
* Installation of 20.04

But had no success, each attempt resulted in an OS that would work momentarily, and then gnome would crash.

## Diagnostics

### Required packages

* curl (for downloading logs and iso files)
* netcat (for shipping logs to [seashells](https://seashells.io))
* genisoimage (to produce bootable bios update USB)
* unzip (to extract zips on the terminal)

To install these on ubuntu:
```bash
sudo apt update && sudo apt install \
genisoimage \
curl \
netcat \
unzip
```

### Logs

__Please make sure to install [Required packages](#required-packages) before following this process.__

During diagnostics on a live usb we ran the following commands to ship logs to [seashells](https://seashells.io):

```bash
# Capture static logs once for later consumption
sudo dmidecode | nc seashells.io 1337
sudo lscpi | nc seashells.io 1337
# Stream logs until machine crashes
sudo dmesg -w | nc seashells.io 1337
sudo journalctl -b -f | nc seashells.io
```

Running any of the above commands will produce output like so:

```bash
$ sudo journalctl -b -f | nc seashells.io 1337
serving at https://seashells.io/v/s5SeWhbp
```

The url it posts in stdout can visited later for log capture - logs should be captured as soon as possible as the links are not permanent. In order to save logs to a file, substitue the _/v/_ in the url to a _/p/_ and curl it to a file:

```bash
curl https://seashells.io/p/s5SeWhbp > journalctl.log
```

* [dmesg.log](./logs/dmesg.log)
* [dmidecode.log](./logs/dmidecode.log)
* [journalctl.log](./logs/journalctl.log)
* [lspci.log](./logs/lspci.log)

### Confirming hardware Model

To confirm the machine is an L490, we used output from [dmidecode.log](./logs/dmidecode.log). The pertinent excerpt is here:

```
Handle 0x0012, DMI type 1, 27 bytes
System Information
	Manufacturer: LENOVO
	Product Name: 20Q5S0M600
	Version: ThinkPad L490
	Serial Number: PG01TCJ4
	UUID: 8dd060cc-26f9-11b2-a85c-f3a05d5236c9
	Wake-up Type: Power Switch
	SKU Number: LENOVO_MT_20Q5_BU_SMB_FM_ThinkPad L490
	Family: ThinkPad L490
```

### Total disk wipe

_This is an excerpt from a comment I made on the original [thread](https://www.reddit.com/r/linuxquestions/comments/g88hhs/need_help_installing_ubuntu/)._

1. Launch a live usb
2. Launch a terminal and run “sudo gdisk -l” and find and note down the name of your main disk, use this in step 2
3. On the assumption your disk is “/dev/nvme0n1” (your screenshots suggest this is the device name) run “sudo gdisk /dev/nvme0n1”
4. In gdisk, press “x” (expert mode)
5. In gdisk, press “z” (zap) and respond yes to all questions- this will remove both gpt and mbr partition tables
6. Quit gdisk
7. Reboot

Your disk should now be as blank as is possible without writing zeroes across the whole disk.

### Firmware Updates

To ensure firmware was not the issue, we chose to attempt a bios + ssd firmware update.

#### Thinkpad L490 BIOS Updates

__Please make sure to install [Required packages](#required-packages) before following this process.__

To confirm what version was currently running we used output from [dmidecode.log](./logs/dmidecode.log). The pertinent excerpt is here:

```
Handle 0x0011, DMI type 0, 26 bytes
BIOS Information
	Vendor: LENOVO
	Version: R0ZET40W (1.18 )
	Release Date: 03/24/2020
	Address: 0xE0000
	Runtime Size: 128 kB
	ROM Size: 32 MB
	Characteristics:
		PCI is supported
		PNP is supported
		BIOS is upgradeable
		BIOS shadowing is allowed
		Boot from CD is supported
		Selectable boot is supported
		EDD is supported
		3.5"/720 kB floppy services are supported (int 13h)
		Print screen service is supported (int 5h)
		8042 keyboard services are supported (int 9h)
		Serial services are supported (int 14h)
		Printer services are supported (int 17h)
		CGA/mono video services are supported (int 10h)
		ACPI is supported
		USB legacy is supported
		BIOS boot specification is supported
		Targeted content distribution is supported
		UEFI is supported
	BIOS Revision: 1.18
	Firmware Revision: 1.15
```
The last two lines indicate the firmware versions, prior to an update they were at 1.11 and 1.12 respectively.

The L490 does not have a CD drive, and yet the only way Lenovo provide to update firmware outside of Windows is a non-USB iso file available (available at time of writing [here](https://download.lenovo.com/pccbbs/mobiles/r0zuj12wd.iso)). Thanks Lenovo.

_Before blindly following this guide, you should ensure you have downloaded the correct iso for your hardware by entering your serial on lenovo's website, and be VERY careful with the dd command listed below_

To make this work with USB we had to utilise a tool called [geteltorino](http://manpages.ubuntu.com/manpages/trusty/man1/geteltorito.1.html) to pull a USB bootable el torino disk image out of the iso, and then write it to USB:

```bash
# Download iso file
curl https://download.lenovo.com/pccbbs/mobiles/r0zuj12wd.iso -o r0zuj12wd.iso
# extract el torino file from iso file
geteltorito r0zuj12wd.iso > bios.img
# On the assumption your usb device is at /dev/sdb (verify with sudo gdisk -l prior to running this command!!!!)
sudo dd if=bios.img of=/dev/sdb bs=1M
# sync disk before unplugging
sync
```

This successfully updated the BIOS, however did not resolve the issue.

#### Intel nvme SSD 790 firmware Updates

_Unfortunately, the same bug that we are trying to fix seemingly causes the linux-based updater to fail. If you have access to another machine with a spare m.2 nvme slot available, however, this step may still be useful in resolving your issue and so will be left here._

__Please make sure to install [Required packages](#required-packages) before following this process.__

Lenovo do provide firmware updates for this SSD, however again they are only available in Windows guise. Intel themselves do, however, provide a linux-friendly USB-Bootable iso image [here](https://downloadcenter.intel.com/download/29248/Intel-SSD-Firmware-Update-Tool). This can be downloaded, extracted, and written to usb:

```bash
### Download zip
curl https://downloadmirror.intel.com/29248/eng/FirmwareUpdateTool_v3_0_8.zip -o FirmwareUpdateTool_v3_0_8.zip
### unzip it
unzip FirmwareUpdateTool_v3_0_8.zip
# On the assumption your usb device is at /dev/sdb (verify with sudo gdisk -l prior to running this command!!!!)
sudo dd if=issdfut_64_3.0.8.iso of=/dev/sdb bs=1m
# sync disk before unplugging
sync
```

#### NVME quirks kernel flag

According to this [bug report](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=910069)

## Sources

### Bug report about Intel 790 SSD on linux
https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=910069
