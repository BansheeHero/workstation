# Fedora Workstations: Silverblue 33 

I am setting up my new personal workstations.
Main purpose is to have a central point to work with:
1. libvirt, KVM and VFIO
1. Operational storage based on LVM
1. Day to day workflows

## Mission statement

Guide to installating Silverblue on a small drive ready for use with further storage.
This means you can separate storage necessary to work from data you work with.
Further it allows me to boot back to a working setup on Fedora 32/33

### Compatibility with your own setup

* Fedora will be mostly compatible. You might have to rewrite things like `rpm-ostree` with `dnf`
* Any Silverblue installation should be compatible with my guides.
* By using containers you can adjust the guide to most modern distros.

### LVM

There are people in love with ZFS and people still preferring mdadm+LVM. I am moving to pure LVM for this setup.
Advantages:
* Flexible support for RAID-like mirror, parity and stripping 
* Thin-provision or for SSDs over-provisioning.

The goal is to have 2 have volume groups:
1. vg_silverblue contains volumes relevant to your distro installation.
2. vg_data contains volumes relevant to you and your data. 
You can also have just one: vg_data and ignore my setup. Even use Silverblue default partitioning.

### Troubleshooting

Each chapter should contain confirmation that you did it correctly.
These commands are not necessary, but you can then identify where it went wrong.

## Sources:

https://docs.fedoraproject.org/en-US/fedora-silverblue/installation/
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/logical_volume_manager_administration/raid_volumes#create-raid

# First test with default partition and encryption.

I used my current workstation to run Silverblue in a virtual machine (VM).
Does not work for me:
* I do not like BTRFS and prefer ext4 or XFS
* Setup is not using LVM

## Custom partiitons

Your root and var directories should be large enough.
Make sure to get all data directories to a separate mount. This will allow you to change them later or move them to `vg_data`.

![Custom partiions](/images/Installation_CustomSetup03.png)

### Small Flash or SSD drive

This one will serve mainly the distro and not data. Data can be moved to it if prefered. Just not shared between.

1. /boot, 1GB, xfs
1. LVM, Encrypted. vg_silverblue, 30GB or rest of the drive.

### Other drives or partiitons

These will be our drives for data. No need to assign mirroring or parity yet.
It is also possible to create this group outside of the installer.

1. LVM, Encrypted or not. vg_data
1. Anything else

### LVMs

We will be using at least the following mounts. I chose to move some from vg_silverblue to the vg_data.
Sizes are not really precise and I have already moved `/home` to `vg_data`.

`vg_silverblue`:
1. lv_root: / 30GB, xfs or ext4
1. lv_log: /var/log 1GB, xfs or ext4
1. lv_swap: swap 1GB, xfs or ext4
1. lv_silverblue_apps: /var/containers 1GB

`vg_data`:
1. lv_home: /var/home 10GB

# More info on LVM

This is where I cover creating and managing volumes:
* Resizing and moving
* Changing redundancy and stripping

## Creating a RAID-like volume

In our current setup we already have a volume we would like to have redundancy on.
Here I am adding mirroring to our home directory/
```
lvconvert -m1 /dev/vg_data/lv_home
```
