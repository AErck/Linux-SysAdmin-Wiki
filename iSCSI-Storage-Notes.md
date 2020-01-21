# ISCSI Terminology

iSCSI -
• Internet Small Computer Interface
• A storage area networking protocol 

LUN - 
• A logical unit number (LUN) represents a single addressable iSCSI disk that is exported from an iSCSI target server.

Target-
• A target server is a server that emulates a backstory and presents it as a LUN to initiators.
• A target LUN is the logical unit itself exported by the target server.

ACL -
• An Access Control List
• ACLs control which clients have access to an iSCSI target.

IQN -
• ISCSI Qualified Name is a unique name used to identify an iSCSI target server
• Every iSCSI target server has an IQN
• Example: iqn:2018-03.com.localnet:maillun
	◦ iqn lets you know it is an IQN
	◦ 2018-03 is the date it was created
	◦ com.localnet is the domain name in reverse
	◦ maillun is a string of characters to differentiate the different targets that may be on this iSCSI target server

Alias -
• An optional string of up to 255 characters describing the target

iSCSI Authentication -
• Authentication is handled by challenge-handshake protocol or CHAP that uses usernames and passwords
	◦ Authentication Modes
		‣ CHAP initiator authentication
			• Only the initiator needs to authenticate
		‣ Mutual CHAP authentication
			• Both the initiator and target need to authenticate to confirm their identities
		‣ Demo Mode
			• No authentication is required (Default Mode)

Backstore - 
• The storage resource that backs the LUN
• The resource may be:
	◦ Entire physical device
	◦ Partition
	◦ RAID device
	◦ LVM logical volume
	◦ File
	◦ RAMDisk

Initiator - 
• A client that accesses the LUNs on a target iSCSI server

iSNS - 
• iSCSI Storage Name Service is an iSCSI protocol used by an initiator to discover shared LUNs

Node - 
• A single discoverable object on an iSCSI SAN

Portal - 
• This is the iSCSI SAN equivalent to a socket in PCP networking
• A combination of an IP address and port
• The default port iSCSI uses is 3260

# Install and set up iSCSI packages

Starting on the rhhost1 device to configure it as the target server
	$ sudo yum install -y targetcli 
Now we need to start the targetcli service
	$ sudo systemctl start target
	$ sudo systemctl enable target
Next we will install the iSCSI client software on rhhost2
	$ sudo yum install -y iscsi-initiator-utils
Then start the iscsid service
	$ sudo systemctl start iscsid
	$ sudo systemctl enable iscsid
We also need to start the iscsi service
	$ sudo systemctl start iscsi
	$ sudo systemctl enable iscsi

# iSCSI addressing

Due to the inconsistent assigning of device names in RHEL (based on order of scanning and reporting in) it is best practice to use the iSCSI World Wide Identifier (WWID) system. If you do not, there is a good chance that the wrong name will be given to a device booting or scanning in out of order, resulting in data corruption.

# Create iSCSI backstores 

We will create a FileIO backstore on rhhost1
Start by accessing the targetcli
	/> sudo targetcli
	Use ls to list all existing objects 
	/> ls
	We should see that we do not have any objects yet
	Now we will actually create the FileIO file that will back this backstore
	/> /backstores/fileio create file1 /temp/disk1.img 200M write_back=false
	To view the new backstore use ls again
	/> ls
We have successfully created the FileIO backstore. More on using it later.

# Create an iSCSI target

To create an iSCSI target we will need to move to the iSCSI directory from the targetcli
	/> cd /iscsi
	Use ls to see a list of targets
	/> ls
	Nothing to see here yet. Type in create and hit enter
	/> create
	targetcli will create a new target with a long name and it will also make a portal group.
	/> ls
	You should be able to see a target, a target portal group, a portal, but no LUNs.
	To make our lives easier, lets create a target with a useful name
	/> create iqn.2018-04.com.localnet:filedisk1
	Go ahead and remove the long named one we first created
	/> delete <long.iqn>

# Create Logical Unit Number (LUN)

Now we need to create a LUN for our FileIO backstore in the targetcli.
	$ sudo targetcli
	Make sure you are in the root
	/> cd /
	Remember to use tab, cd into 
	/> cd /iscsi/iqn.2018-04.com.localnet:filedisk1/tpg1/
	Now create the LUN
	/> luns/ create /backstores/fileio/file1
	Verify with ls
	/> ls

# Configure access control lists

Once again start targetcli
	$ sudo targetcli
	Change to the appropriate directory
	/> cd iscsi/iqn.2018-04.com.localnet:filedisk1/tpg1
	Type pwd to verify your location
	/> pwd
	To create an acl, we will specify the acls/ path and the name of the target and identifier.
	/> acls/ create iqn.2018-04.com.localnet:client1
	Type ls to verify
	/> ls
	Move into the acls directory and verify with pwd
	/> cd acls/iqn.2018-04.com.localnet:client1/
	/> pwd
	Now to set access control, we set a username and password
	/> set auth userid=user1
	/> set auth password=password
	To verify move up two levels
	/> cd ../..
	/> pwd
	Now we can exit targetcli properly to save our configuration 
	/> exit

# Configure Firewalld
	
To make our target functional, we need to allow traffic through the firewall.
To do so, we need to check if the target service is answering on port 3260
	$ sudo netstat -ant
With that confirmed, we will add the rule to firewalld
	$ sudo firewall-cmd --permanent --add-service iscsi-target
Now set the rules into effect
	$ sudo firewall-cmd --reload


# Create an iSCSI initiator

To setup our iSCSI initiator, we need to edit the iSCSI config file on the client computer (rhhost2)
	$ sudo vim /etc/iscsi/initiatorname.iscsi
	Edit the InitiatorName line to
	InitiatorName=iqn.2018-04.com.localnet:client1
Now we edit a second file
	$ sudo vim /etc/iscsi/iscsid.conf
	Uncomment the following lines and change them to match
	node.session.auth.authmethod = CHAP
	node.session.auth.username = user1
	node.session.auth.password = password
We need to restart the iscsi and iscsid processes
	$ sudo systemctl restart iscsi
	$ sudo systemctl restart iscsid
Now lets discover the iscsi target
	$ sudo iscsiadm --mode discovery --type sendtargets --portal <target.ip.addr> (example 192.168.1.86)
Now attempt to login
	$ sudo iscsiadm --mode node --targetname iqn.2018-04.com.localnet:filedisk1 --portal <target.portal.ip.addr> (example 192.168.1.86:3260) --login
You will get feedback letting you know of success
Now check the local iscsi storage with lsblk
	$ sudo lsblk --scsi
	If working, we should see device with vendor of LIO-ORG. Note the name of the device, sdb in this example.
Check to see if the iscsi device is in read-only mode, use the correct device name here
	$ sudo lsblk | egrep “NAME | sdb”
	If the RO column is 1, it is in read-only mode. We want to see a 0 here.
The device is now ready to be mounted as a local drive.

# Partition and mount by UUID

To get started, verify that we have access to the iscsi device
	$ sudo lsblk --scsi
 Next we need to add a partition to the device and format it
	$ sudo gdisk/dev/sdb
	n
	default partition num
	default first sector
	default last sector
	default partition type
	w to write the partition
	y to proceed
Verify with lsblk
	$ lsblk
	You should see /dev/sdb1
Now we will format it with ext4 file format
	$ sudo makes.ext4 /dev/sdb1
Now get the UUID of the filesystem 
	$ sudo blkid /dev/sdb1
	Copy the UUID
Create the mount point for the device
	$ sudo mkdir /mnt/filedisk1
	It makes your life easier to use the name of the target or target identifier for the name of the directory. This will help you associate the target with local directories.
Now we will mount the device.
	$ sudo mount <past-uuid> /mnt/filedisk1
Verify successful mount with lsblk
	$ lsblk
Now add the entry to the etc/fstab
	$ sudo vim /etc/fstab
	Add the following line to the end of the file
	<past-uuid> /mnt/filedisk1 ext4 _netdev 0 0
Test this by unmounting and auto mounting it again
	$ sudo umount /mnt/filedisk1
	$ sudo mount -a
Check that it worked with lsblk. If you see it there mounted again all is working well.

VERY IMPORTANT NOTE!!!!
Before you shutdown either machine, you MUST shutdown the initiator first, or the initiator needs to log out of the target. If the target is shutdown first, it will yank the device off the mount point from under the initiator causing the initiation to be unable to reboot.

# Remove a device from a target

First unmount the device from the initiator (rhhost2)
	$ sudo umount /mnt/filedisk1

Now we need to remove it from ALL configuration files, in the example case, we only have to worry about /etc/fstab
	$ sudo vim /etc/fstab
	Delete the line about the device and save.
Next we logout of the target
	$ sudo iscsiadm --mode node --targetname iqn.2018-04.com.localnet:filedisk1 --portal <target.portal.ip.addr> --logout
Verify with lsblk and /sys/block
Next we log into the iscsi target (rhhost1) and remove the device there from the targetcli
	$ sudo targetcli
	/>  cd /
	/> ls
	/> /iscsi/iqn.2018-04.com.localnet:filedisk1/tpg1/acls/ delete iqn.2018-04.com.localnet:client1
Verify with ls
	/> ls
Now we need to remove the rest of the components, but we can do it in a shorter way.
	/> /iscsi/ delete iqn.2018-04.com.localnet:filedisk1
Verify with ls
	/> ls
We should see all of the target components are gone except the backstore
	/> /backstores/fileio/ delete file1
	/> ls
To save our changes and to get back to BASH, use the exit command
	/> exit 
