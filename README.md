# Lenovo Thinkpad L490 + Intel nvme SSD 760p

## Original issue

The original issue was posted on reddit by [/u/Kronikaetor](https://www.reddit.com/user/Kronikaetor/) and is available [here](https://www.reddit.com/r/linuxquestions/comments/g88hhs/need_help_installing_ubuntu/).

## Summary

Kronikaetor has an in-warranty Thinkpad L490. Initially shipped with freedos, he installed Ubuntu 19.04 until it reached EOL, then attempted to install 20.04 at which point his laptop refused to run stably. He attempted:

* Installation of 18.04
* Installation of 20.04

But had no success, each attempt resulted in an OS that would work momentarily, and then gnome would crash.

## Diagnostics

### Logs

During diagnostics on a live usb we ran the following commands to ship logs to [seashells](https://seashells.io):

```bash
# Install netcat to perform log-shipping
apt install netcat
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
