[aerck@onyx ~]$ lsblk -f
NAME            FSTYPE      LABEL UUID                                   MOUNTPOINT
sda                                                                      
├─sda1          xfs               179b4d6b-abb7-422c-b502-11a50404a895   /boot
└─sda2          LVM2_member       dXKr2l-0tUL-qHMe-n0Ys-ZiSf-jaC7-kHIkou 
  ├─centos-root xfs               831eeebe-1119-4106-9732-b5c2c44a6a6e   /
  ├─centos-swap swap              17a57a6f-1f20-4bb2-8209-ce14775f6f02   [SWAP]
  └─centos-home xfs               10da2f5a-6c79-4578-9480-21c01f6a740e   /home
sdb                                                                      
├─sdb1          xfs               350e3d90-63d2-494d-9d7b-cc7c78fa20fc   /export
└─sdb2          xfs               5adc6f0d-b096-40f6-982d-e4753c3346f4   /home
sr0 


[aerck@onyx ~]$ df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root   45G   23G   22G  51% /
devtmpfs                 3.9G     0  3.9G   0% /dev
tmpfs                    3.9G     0  3.9G   0% /dev/shm
tmpfs                    3.9G  401M  3.5G  11% /run
tmpfs                    3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/sdb2                466G  1.3G  465G   1% /home
/dev/sda1               1014M  190M  825M  19% /boot
/dev/sdb1                466G  4.4G  462G   1% /export
tmpfs                    789M     0  789M   0% /run/user/1001


[aerck@onyx ~]$ sudo vgs
  VG     #PV #LV #SN Attr   VSize  VFree
  centos   1   3   0 wz--n- 73.50g    0 

Based on initial investigation, /home and /export are not LVM partitions. This will make expansion difficult, as I will need to backup the data (as originally planned) prior to moving the sdb1 (/export) and sdb2 (/home) into a volume group to be managed with LVM.
# Creating Partitions
Using lsblk, we can see all of the known block devices and their partitions.
To modify partitions, you can use fdisk, gdisk, or parted. The first to have a sfdisk and cfdisk for scripts and use with a curses like interface respectively.
First example we will use gdisk
	$ sudo gdisk /dev/sdb
Then you will interactivity use gdisk to create, modify, or delete partitions.
To make sure the parts of the system are aware of new partitions that were created or modified, use partprobe
	$ sudo partprobe
Creating and modifying partitions with parted, it may need to be installed.
	$ sudo parted

#steps for creating new LVM storage system using LVM tools
Unmount the drive you are going to convert to LVM. 
	$ sudo umount /dev/sdb1
Create Physical Volume with pvcreate (this will destroy files on /dev/sdb1) 
We can use entire drive /dev/sdb rather than /dev/sdb1, special instructions required for SSDs
	$ sudo pvcreate /dev/sdb1
Verify successful creation of the physical volume with pvs 
	$ sudo pvs
	/dev/sdb1 should be listed
Now create the Volume Group with vgcreate, this can accept multiple Physical Volumes at once
	$ sudo vgcreate <vg-name> <pv-1> <pv-2> ....
	Example $ sudo vgcreate vgdata /dev/sdb1
Verify with vgs 
	$ sudo vgs
	vgdata should be listed if you used the example command above
Next we create a Logical Volume in the Volume Group we just created with lvcreate 
	$ sudo lvcreate --name <lv-name> --size <lv-size> <volume-group-name>
	Example $ sudo lvcreate --name lvdata -- size 500M vgdata
Verify with lvs
	$ sudo lvs
	lvdata should be listed.
Logical Volumes can be referred to by two different methods:
	1. /dev/VolumeGroup/LogicalVolume
	2. /dev/mapper/VolumeGroup-LogicalVolume
It may be necessary	to “activate” the Logical Volume to get it to “show-up” in the system. It will not hurt anything to run the following command even if it is not needed.
	$ sudo vgchange -ay
Now we need to format the Logical Volume with a file system, we will use xfs due to its usefulness with HPC
	$ sudo makes -t xfs /dev/vgdata/lvdata
Verify the previous step with blkid
	$ sudo blkid
	The Logical Volume will be listed at the end with its file system type
The Logical Volume with XFS can now be mounted
If you need to create a mount point, use mkdir 
	$ sudo mkdir /media/lvdata
Mount the Logical Volume 
	$ sudo mount /dev/vgdata/lvdata /media/lvdata
Verify the mount with df and lsblk 
	$ df -h
	$ lsblk
	You should see the Logical Volume listed with its size
If we need to expand an existing Volume Group use vgextend 
	$ sudo vgextend <VolumeGroup> <PhysicalVolume>
You will also then need to expand the Logical Volume of choice in the Volume Group
	$ sudo lvresize -l 100%FREE /dev/vgdata/lvdata
Next you will then need to expand the file system to utilize the new space
	Verify this command with man xfs_growfs
	$ sudo xfs_growfs /dev/vgdata/lvdata
Verify the new file system size with df -h
	$ df -h
You can run a file system check to look for errors in the volume using e2fsck
	$ sudo e2fsck -ff /dev/vgdata/lvdata
	There will be 5 checks that need to be passed
lvresize can be used to unmount, resize, and remount a logical volume in one step
	$ sudo lvresize -r -L <new-volume-size> /dev/vgdata/lvdata

# System Storage Manager (SSM) to work with volumes and drives
Install SSM if needed
	$ sudo yum install -y system-storage-manager
You can list available devices with $ sudo ssm list
SSM Pools are equivalent to LVM Volume Groups
Create a new Pool, Volume, and File System with
	$ sudo ssm create --fs xfs -s <pool-size> -p <pool-name> -n <volume-name> <partition-1> <partition-2> ...
Example $ sudo ssm create --fs xfs -s 1G -p ssmpool -n ssmvol /dev/sdb1 /dev/sdc1
SSM can use it’s “check” function to run a file system check on any storage pool regardless of file system type.
	$ sudo ssm check /dev/ssmpool/ssmvol
Likewise, SSM has tools like ssm resize, ssm snapshot, ssm remove, and ssm mount all with similar abstracted usability like ssm check.

# Mounting file systems at boot by ID or Label
For the example we use partition /dev/sdd1 (ext4) and volume /dev/ssmpool/ssmvol (xfs)
First find the UUID of ssmvol with blkid
	$ sudo blkid
	it will look like UUID=“...”, copy it
Edit the /etc/fstab file
	$ sudo vi /etc/fstab
	Add a line for ssmvol that looks like this:
	UUID=“..”	/media/ssmvol	xfs		defaults				0 0
Next we will mount the /dev/sdd1 partition by Label, which is File System Dependant 
	ext4 uses e2label and xfs uses xfs_admin
	$sudo e2label /dev/sdd1 backups
Verify with e2label
	$ sudo e2label /dev/sdd1
Add a line in /etc/fstab for /dev/sdd1
	LABEL=backups	 /media/sdd1	 ext4	 defaults	 0 0	
Verify with mount and df
	$ sudo mount -a
	$ df
# Backup and restore an EXT file system
Run a file check on the file system to see if there are any errors prior to backup starting.
	$ sudo ssm check <file-system-path>
	Example $ sudo ssm check /dev/ssmpool/ssmvol
Create an SOS report on the system configuration
	$ sudo sosreport
	Use the defaults for hostname and case ID.
	This report is stored in /var/tmp 
Next we will create a backup with the dump command, which we will need to install.
	$ sudo yum install -y dump
Run the dump command as follows
	$ sudo dump -0uf /home/<volume-dumped>.dump <path-of-volume-to-dump>
	Example $ sudo -0uf /home/lvraid.dump /dev/vgraid/lvraid 
Dump can be piped through ssh to save a dump file of a local system on a remote host.
Since this will be restored to the existing drive, we format the volume and restore the dump file to fill in the drive.
	Overwrite the volume
	$ sudo mkfs.ext4 /vgraid/lvraid
	Create a mount point of /media/restore
	$ sudo mkdir /media/restore
	Now mount the formatted volume to the restore point
	$ sudo mount /dev/vgraid/lvraid /media/restore
	Move to the restore point and run the restore command
	$ cd /media/restore
	$ sudo restore -rf /home/lvraid.dump

# Archiving and Compressing files
With RHEL we will utilize Tar for archiving files and their metadata 
Our example will to be create an archive of /etc.
	$ sudo tar --xattrs -cvpf etc.tar /etc
Now we will create three compressed versions of this file with GZip, BZip2, and XZ
	$ sudo tar --xattrs --gzip -cvpf etc.tar.gz /etc
	$ sudo tar --xattrs --bzip2 -cvpf etc.tar.bz2 /etc
	$ sudo tar --xattrs --xz -cvpf etc.tar.xz /etc
We can extract the tar file with
	$ sudo tar --xattrs -xvpf etc.tar -C <directory-to-extract-into>

