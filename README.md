# User Guide

## Description
The purpose of this project is to create a LUKS encrypted Ubuntu AWS EC2 instance.

Target system will have the following configuration:
``````
NAME              MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
xvda              202:0    0    5G  0 disk
├─xvda1           202:1    0  127M  0 part  /boot
└─xvda2           202:2    0  4.9G  0 part
  └─lvm_crypt     252:0    0  4.9G  0 crypt
    ├─system-swap 252:1    0  512M  0 lvm   [SWAP]
    └─system-root 252:2    0    4G  0 lvm   /
``````

## Installation

#### Requirements
- Check for open bugs [here](https://github.com/dzaninov/aws2luks/issues?q=is%3Aissue+is%3Aopen+label%3Abug)
- Create new Ubuntu work instance or use the existing **non-production** instance.
- Install awscli and jq.
``````
sudo apt-get install awscli jq
``````

#### Downloading the files
``````
git clone https://github.com/dzaninov/aws2luks.git
``````

#### Configuration
``````
vi aws2luks.conf    # review the configuration
vi aws2luks.custom  # review the custom script
``````

#### IAM Policy
Script needs permissions to run aws cli commands.

Generate IAM policy
``````
./policy aws2luks
``````
Add policy as customer managed policy to IAM.

## Creating the instance
Create the encrypted instance and start it.
``````
sudo su
 export LUKS_PASSWORD=unlock_password
 export AWS_ACCESS_KEY_ID=iam_access_key_id
 export AWS_SECRET_ACCESS_KEY=iam_secret_access_key
./aws2luks
``````
Space before export is required for bash not to save the command in history.

Instead of exporting AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY stored credentials can be used.

If KEEP_IMAGE is set AMI will stay behind so you can create more instances from it.

## Sample output
``````
# ./aws2luks
Creating a volume
Seeding encrypted device with random data
Downloading the OS image
######################################################################## 100.0%
Writing the OS image
Setting up the OS
Creating snapshot (very slow)
Registering image
Creating Instance
Instance i-00a23999ac234acee is running
SSH to 52.179.190.207 as root to boot the system
``````

## Booting
SSH to instance as **root** to unlock the LUKS device and continue booting.
``````
Using username "root".
Authenticating with public key "ssh"
Passphrase for key "ssh":


BusyBox v1.22.1 (Ubuntu 1:1.22.0-15ubuntu1) built-in shell (ash)
Enter 'help' for a list of built-in commands.

Enter 'unlock' to unlock and continue booting
Enter 'rescue' to enter the rescue shell

# unlock
Enter passphrase for /dev/xvda2:

``````
SSH connection will be dropped when system starts booting.

## Rescue shell
If the system is not usable after it boots or it won't boot, the rescue shell can be started to fix the issue.
``````
# rescue
Enter passphrase for /dev/xvda2:

Exit the rescue shell when done

root@ip-122-35-27-182:/# chmod -x /etc/init.d/ufw
root@ip-122-35-27-182:/# exit
exit

Exited rescue shell

#
``````
The above example shows how to disable the firewall in case it prevents the SSH login.

No drivers are loaded at this time so some commands will have issues but any file can be modified.

## Configurable options
Options that can be configured in aws2luks.conf.

#### OS source options
- **OS_URL** - URL of the latest stable Ubuntu release
- **OS_IMAGE** - Ubuntu image name within the OS_URL archive
- **OS_SHA256** - URL to file containing SHA256 hashes where OS_URL archive can be found

#### Cloud options
- **INSTANCE_TYPE** - Instance type to create
- **VOLUME_TYPE** - Instance volume type
- **SSH_KEY** - SSH key pair name configured for EC2 Region
- **SECURITY_GROUP** - Network security group
- **KEEP_IMAGE** - Keep resulting AMI image and the snapshot after the instance is created

#### Sizing
Remember to leave some space in the volume group for snapshots or additional encrypted volumes.
Any volumes added later will be automatically encrypted.
- **TARGET_SIZE_GB** - Size of the target volume
- **BOOT_SIZE_MB** - Size of the /boot filesystem
- **SWAP_SIZE_MB** - Swap size.  Set to 0 for no swap.
- **ROOT_SIZE_MB** - Root filesystem size

#### Encryption
- **LVM_DEVICE** - Name of the plaintext device hosting the LVM. Can't exist on the work system.
- **RANDOMIZE** - Seed encrypted device with random data

#### LVM naming
These can't exist on the work system.
- **VG_NAME** - Name of the LVM volume group
- **LV_NAME_SWAP** - Name of the swap LVM logical volume.
- **LV_NAME_ROOT** - Name of the root LVM logical volume.

#### Device label naming
These labels can't exist on the work system.
- **BOOT_LABEL** - /boot filesystem label
- **SWAP_LABEL** - swap device label
- **ROOT_LABEL** - root filesystem label

#### Other
- **BOOT_FS_TYPE** - /boot filesystem type
- **WORK_DEVICE** - Device to which to attach the volume while working on it. This device must not be currently in use.
- **WORK_DIR** - Mount point for the WORK_DEVICE. It will be created
- **CUSTOM_SCRIPT** - Custom script to call within chroot'ed instance volume
