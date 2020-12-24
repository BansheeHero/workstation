# Silverblue 33 Fedora Workstations

## Mission statement

Guide to installating SilverBlue on a small drive ready for use with further storage.
Similar to ESXi or Live Distrios.

### Compatibility with your own setup

* Fedora will be mostly compatible.
* You can install any silverblue setup, I will try to keep the later guides independent from my setup.
* By using containers you can adjust the guide to most modern setups.

### LVM

LVM was chosen for offering the most features without sacrificing flexibility.
The goal is to have 2 have volume groups:
1. vg_silverblue contains volumes relevant to your distro.
2. vg_data contains volumes relevant to you and your data.
You can also have just one: vg_data and ignore my setup. Even use Silverblue default partitioning.

### Troubleshooting

Each chapter should contain confirmation that you did it correctly.
These commands are not necessary, but you can then identify where it went wrong.

## Sources:

https://docs.fedoraproject.org/en-US/fedora-silverblue/installation/

# First test with default partition and encryption.

Does not work for me: BTRFS, no LVM.
Now that does not mean you have to follow.

## Custom partiitons

Room has to be large due to Installer not recognizing Silverblue structure (5GB).
Make sure to get homes, containers and other /var directories mounted separately.

![Custom partiions](/images/CustomPartitions.png)

Small drive:
/boot, 1GB
LVM, Encrypted. vg_silverblue
Other drives:
LVM, Encrypted or not. vg_data

vg_silverblue::
1. lv_root: / 30GB, xfs or ext4
2. lv_log: /var/log 1GB, xfs or ext4
3. lv_swap: swap 1GB, xfs or ext4
vg_data:
4. lv_home: /var/home 1GB
5. lv_silverblue_apps: /var/containers 1GB

# LVM

LVM will be used for everything, including mirror or stripping.
There should be at least 2 Volume Groups - System and Data.
This does complicate home and containers folders, but it allows us to completely separate USB from the datta drive.

## Creating a RAID-like volume


```
lvcreate --type raid1 -m 1 -L 1G -n lv_homes vg_data
```
