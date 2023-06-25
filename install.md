## Hardware

Internal description/pictures:

*  https://www.juniper.net/documentation/en_US/release-independent/junos/topics/reference/compliance/srx300-letter-of-volatility.pdf

This device probably has problems with ATP eUSB modules becoming slower over time:

```
Octeon srx_300(ram)# usb storage
  Device 0: Vendor: ATP      Rev: 1100 Prod: ATP CG eUSB     
            Type: Hard Disk
            Capacity: 7672.0 MB = 7.4 GB (15712256 x 512)
```

*  https://www.reddit.com/r/Juniper/comments/pu5jsi/srx300_known_eusb_alternatives_featuring_probably/
*  https://www.reddit.com/r/Juniper/comments/srk4io/dead_flash_in_srx320/

```
loader> nextboot
Platform: srx-sword
    eUSB
    usb
```

## Software Versions/Hashes

From [4](https://www.reddit.com/r/homelab/comments/14iduq6/recovering_an_srx300_with_wiped_partitions/)

If you have an older loader, you cannot install versions later than 15 directly.  You have to install 15 then upgrade to 19 etc.  Upgrading the loader using TFTP may also be an option.

From [4](https://supportportal.juniper.net/s/article/Junos-Software-Versions-Suggested-Releases-to-Consider-and-Evaluate) and https://support.juniper.net/support/downloads/?f=srx

*  23.1.R1 (23 Mar 2023): a4fa09f5bfea8ff25de56a54a67e9b4652db13e601a4f8f823a2fc1dce632546
*  21.4R3-S3 (11 April 2023) (Recommended): 41caa7b5c7fac3a6ba75b31f857ac7c3ec033609f8b93b04f49cc7b9160398b9 (hash listed at https://forums.servethehome.com/index.php?threads/srx300-firmware.29945/page-2)
*  21.4R3 (8 Sep 2022): e1ff3dad1d4ba3b69ad2bc502e512e832b3c4f1682be7b96a7b2a7cc57da2d9d
*  21.2R3 (29 Mar 2022): 1ef777147e281ea447c328a5a792b50f70d7128e5e3e200b792739970752d579

The 15.1X49D90.7 version from archive.org has this SHA256 hash: 154926943a4941eeab918d4fedbd0e57ef8028f849af02ee10aee6986e8a53ce

Devices after June 14 2019 Require at least 15.1X49 due to upgraded eUSB driver/hardware:

*  https://www.reddit.com/r/Juniper/comments/dfi7n6/psa_srx3xx_received_rma_or_purchased_after_june/
*  https://imgur.com/a/p3poySz

* `show chassis hardware detail`
  * ATP CG eUSB are the old modules
  * USB Flash Module are the new modules

### Packages

From [6](https://supportportal.juniper.net/s/article/Junos-Understanding-Junos-software-components-and-installation-package-names):

package-name-m.nZx-distribution.tgz
* package-name is the name of the package; example:  jinstall-ex, jbundle, and jroute, jpfe.
* m.n is the software release, where m represents the major release number; example:  9.0.
* Z indicates the type of software release; example: R indicates released software, and B indicates beta-level software.
* x represents the version of the software release; example: 2.
* distribution indicates the area for which the software package is provided:
* domestic - For the United States and Canada
* export - For worldwide distribution

From [9](https://nsrc.org/wrc/data/2004/873961028427f0b66f130a/junos_sanog.pdf):

1. `show system software`

Packages downloaded from Juniper then installed with `request system software add {name}`

## TFTP Install

From [1](https://www.juniper.net/documentation/us/en/software/junos/junos-install-upgrade/topics/topic-map/install-software-on-srx.html), [8](https://supportportal.juniper.net/s/article/SRX-How-to-install-software-using-TFTPBoot-method-on-SRX300-series):

1. `cp junos...tgz /srv/tftp/`
2. `mkdir /srv/tftp/tmp; cd /srv/tftp`
3. `tar xvf ../junos...tgz`
4. `cp application-pkg.tgz ..`
4. `cp initrd.cpio.gz ..`

1. Press space to abort autoboot and enter U-Boot (`=>`).
2. Set variables for networking:
   * `setenv ipaddr 192.168.1.2`
   * `setenv netmask 255.255.255.248`
   * `setenv gatewayip 192.168.1.1`
   * `setenv serverip 192.168.1.1`
   * `saveenv`
3. `reset` to reboot.
4. Press space bar to access the loader prompt (`loader>`).
5. Plug network into **first** ethernet interface.
   * [8] says to use the FXP interface.
6. `install tftp://192.168.1.1/junos-srxme-11.1R1-domestic.tgz`
   * Enables Ethernet interfaces
   * Downloads the package
   * Installs the package

20 minutes to install.

### TFTP Upgrading u-boot and loader

From [10](https://supportportal.juniper.net/s/article/How-to-Upgrade-u-boot-and-loader-on-SRX-Branch-Devices-using-TFTP-method):

1. Put `u-boot-crc.bin` and `loader_crc` onto the TFTP server.
2. Reboot the device.
3. Press space to abort autoboot and enter U-Boot (`=>`).
4. Set variables for networking if required.
5. `=> ping 192.168.1.1`
6. `=> tftpboot 0x100000 u-boot-crc.bin`
7. `=> bootloader upgrade u-boot active 0x100000`
8. `=> reset`
9. Press space to abort autoboot and enter U-Boot (`=>`).
10. `=> tftpboot 0x100000 loader_crc`
11. `=> bootloader upgrade loader 0x100000`
12. `=> reset`

## USB Install

From [2](https://supportportal.juniper.net/s/article/Size-of-USB-drives-recommended-for-SRX-device-upgrade-downgrade):

* USB 2.0
* FAT/FAT32
* 4-32GB

### Preparation

From [1](https://www.juniper.net/documentation/us/en/software/junos/junos-install-upgrade/topics/topic-map/install-software-on-srx.html):

1. Copy the upgrade image (junos-srxme\*) to the USB drive.
2. Echo ' ' > autoinstall.conf.
3. vi junos-config.conf (optional) (see above).

### Installation (auto, from JunOS)

From [1](https://www.juniper.net/documentation/us/en/software/junos/junos-install-upgrade/topics/topic-map/install-software-on-srx.html), [3](https://supportportal.juniper.net/s/article/SRX-USB-autoinstallation-on-SRX-branch-platforms-for-software-upgrade-downgrade)

1. Insert USB drive.
2. Power on the device.  The LEDs should blink amber.
3. Press "Reset Config" button.  The LEDs will glow solid amber.
4. Green LEDs indicate success.
5. Remove USB drive.

Only works if `set autoinstallation usb disable` is not set (from JunOS?).

### Installation (manual, from loader)

From [1](https://www.juniper.net/documentation/us/en/software/junos/junos-install-upgrade/topics/topic-map/install-software-on-srx.html):

1. Insert USB drive.
2. Power on the device.
3. Press space bar to access the loader prompt (`loader>`).
4. `install file:///junos-srxme-11.1R1-domestic.tgz`

May require `set currdev=disk1` where disk name is learnt from `lsdev`.

If you see `Target device selected for installation: internal media cannot open package (error 2).`
then power cycle and try again.

From [12](https://www.reddit.com/r/Juniper/comments/dzuw4u/fail_to_boot_juniper_srx340_by_usb/):

If you see `error 22: cannot open package (error 22)` then RMA.

## JunOS

### Log in

From [9](https://nsrc.org/wrc/data/2004/873961028427f0b66f130a/junos_sanog.pdf):

1. `login: root`
2. `# cli`
3. `root@host> configure`
4. `root@host#`

### Check status

From [1](https://www.juniper.net/documentation/us/en/software/junos/junos-install-upgrade/topics/topic-map/install-software-on-srx.html), [10](https://supportportal.juniper.net/s/article/How-to-Upgrade-u-boot-and-loader-on-SRX-Branch-Devices-using-TFTP-method):

1. `show system`
1. `show system alarms`
2. `show version`
3. `show configuration`

### Configure for two alternate boot partitions

From [7](https://www.reddit.com/r/Juniper/comments/le18qg/srx300_cant_boot_after_power_outage/):

1. `request system snapshot slice alternate`
2. `edit` (or configure?)
   1. `set system autosnapshot`
   3. `commit`

From [9](https://nsrc.org/wrc/data/2004/873961028427f0b66f130a/junos_sanog.pdf):

1. `request system snapshot`

### Backup to USB

From [5](https://supportportal.juniper.net/s/article/SRX-Getting-Started-Junos-Software-Installation-Upgrade):

1. `user@host> start shell user root`
2. `ls /dev/da*`
3. `mkdir /var/tmp/usb`
4. `mount_msdosfs /dev/da2s1 /var/tmp/usb` or `mount -t msdosfs /dev/da2s1 /var/tmp/usb`
5. `request system snapshot media usb`
6. `umount /var/tmp/usb`

### Set root password

From [9](https://nsrc.org/wrc/data/2004/873961028427f0b66f130a/junos_sanog.pdf):

1. `root@> configure` (same as edit?)
   1. `set system root-authentication plaintext-password`
   2. `set system root-authentication ssh-rsa {key}`
   3. `set system host-name lab1`
   4. `set system domain-name i8c.net`
   5. `set system name-server 8.8.8.8`
   6. `commit`

### IP Configuration

From [9](https://nsrc.org/wrc/data/2004/873961028427f0b66f130a/junos_sanog.pdf):

1. `root@> configure` (same as edit?)
   1. `set interfaces fxp0 unit 0 family inet address 192.168.0.2/24`
   2. `set system backup-router 192.168.0.1`
   3. `set routing-options static route default nexthop 192.168.0.1 retain no-advertise`
   4. `commit`

From [11](https://www.juniper.net/documentation/us/en/software/junos/junos-install-upgrade/topics/topic-map/understanding-autoinstallation-config-files.html):

1. `edit system services netconf ssh`
2. `edit system services web-management https interface [irb.0]`

### Reboot

From [1](https://www.juniper.net/documentation/us/en/software/junos/junos-install-upgrade/topics/topic-map/install-software-on-srx.html), [5](https://supportportal.juniper.net/s/article/SRX-Getting-Started-Junos-Software-Installation-Upgrade):

* `user@host> request system reboot`
* `user@host> request system reboot at 5 in 50 media internal message stop`

### Shutdown

From [1](https://www.juniper.net/documentation/us/en/software/junos/junos-install-upgrade/topics/topic-map/install-software-on-srx.html):

1. `user@host> request system halt at now`

### Upgrade

1. `request system software add no-copy /var/tmp/junos-srxme-11.1R1-domestic.tgz`

### Upgrade bootloader

From [1](https://www.juniper.net/documentation/us/en/software/junos/junos-install-upgrade/topics/topic-map/install-software-on-srx.html):

1. Have latest versions in `/boot/uboot` and `/boot/loader`.
2. `bootupgrade -u /boot/uboot -l /boot/loader`
3. `show system firmware`
4. `show chassis routing-engine bios`

### Purge logs

1. `show system storage`
2. `show log messages`
3. `request system storage cleanup`

## Autoconfig files

### autoinstall.conf

From [11](https://www.juniper.net/documentation/us/en/software/junos/junos-install-upgrade/topics/topic-map/understanding-autoinstallation-config-files.html):

1. DHCP provides hostname and TFTP server address.
   1. If not hostname is provided, tftp://network.conf is consulted (host/IP map).
2. TFTP server provides config in tftp://{hostname}.conf
   1. Default is tftp://router.conf if no hostname is found.

Autoinstall.conf looks like it's possible the same as router.conf/{hostname}.conf/junos-config below?

### junos-config.conf

This looks like a structured version of the CLI config commands.

From [1](https://www.juniper.net/documentation/us/en/software/junos/junos-install-upgrade/topics/topic-map/install-software-on-srx.html):

```
system {
    host-name host-1;
    domain-name example.net;
    name-server 8.8.8.8;
    domain-search [ abc.example.net example.net device1.example.net ];
    root-authentication {
        encrypted-password "$ABC123"; ## SECRET-DATA
    }
}
...
routing-options {
    static {
        route 0.0.0.0/0 next-hop 10.207.31.254;
    }
}
```

## References

1. https://www.juniper.net/documentation/us/en/software/junos/junos-install-upgrade/topics/topic-map/install-software-on-srx.html
2. https://supportportal.juniper.net/s/article/Size-of-USB-drives-recommended-for-SRX-device-upgrade-downgrade
3. https://supportportal.juniper.net/s/article/SRX-USB-autoinstallation-on-SRX-branch-platforms-for-software-upgrade-downgrade
4. https://supportportal.juniper.net/s/article/Junos-Software-Versions-Suggested-Releases-to-Consider-and-Evaluate
5. https://supportportal.juniper.net/s/article/SRX-Getting-Started-Junos-Software-Installation-Upgrade
6. https://supportportal.juniper.net/s/article/Junos-Understanding-Junos-software-components-and-installation-package-names
7. https://www.reddit.com/r/Juniper/comments/le18qg/srx300_cant_boot_after_power_outage/
8. https://supportportal.juniper.net/s/article/SRX-How-to-install-software-using-TFTPBoot-method-on-SRX300-series
9. https://nsrc.org/wrc/data/2004/873961028427f0b66f130a/junos_sanog.pdf
10. https://supportportal.juniper.net/s/article/How-to-Upgrade-u-boot-and-loader-on-SRX-Branch-Devices-using-TFTP-method
11. https://www.juniper.net/documentation/us/en/software/junos/junos-install-upgrade/topics/topic-map/understanding-autoinstallation-config-files.html
12. https://www.reddit.com/r/Juniper/comments/dzuw4u/fail_to_boot_juniper_srx340_by_usb/
13. https://www-origin-stage.junipercloud.net/documentation/us/en/software/junos/junos-install-upgrade/junos-install-upgrade.pdf
14. https://www.reddit.com/r/homelab/comments/14iduq6/recovering_an_srx300_with_wiped_partitions/
