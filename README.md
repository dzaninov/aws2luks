# User Guide

## Description
The purpose of this project is to create LUKS encrypted EC2 instance in the AWS cloud.

## Target configuration
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

## Configurable options
Options that can be configured in aws2luks.conf.

### OS source options
- **OS_URL** - URL of the latest stable Ubuntu release
- **OS_IMAGE** - Ubuntu image name within the OS_URL archive
- **OS_SHA256** - URL to file containing SHA256 hashes where OS_URL archive can be found

### Cloud options
- **INSTANCE_TYPE** - Instance type to create
- **VOLUME_TYPE** - Instance volume type
- **SSH_KEY** - SSH key pair name configured for EC2 Region
- **KEEP_IMAGE** - Keep resulting AMI image and the snapshot after the instance is created

### Sizing
Remember to leave some space in the volume group for snapshots or additional encrypted volumes.
Any volumes added later will be automatically encrypted.
- **TARGET_SIZE_GB** - Size of the target volume
- **BOOT_SIZE_MB** - Size of the /boot filesystem
- **SWAP_SIZE_MB** - Swap size
- **ROOT_SIZE_MB** - Root filesystem size

### Encryption
- **LVM_DEVICE** - Name of the plaintext device hosting the LVM. Can't exist on the work system.
- **RANDOMIZE** - Seed encrypted device with random data

### LVM naming
These can't exist on the work system.
- **VG_NAME** - Name of the LVM volume group
- **LV_NAME_SWAP** - Name of the swap LVM logical volume.
- **LV_NAME_ROOT** - Name of the root LVM logical volume.

### Device label naming
These labels can't exist on the work system.
- **BOOT_LABEL** - /boot filesystem label
- **SWAP_LABEL** - swap device label
- **ROOT_LABEL** - root filesystem label

### Other
- **BOOT_FS_TYPE** - /boot filesystem type
- **WORK_DEVICE** - Device to which to attach the volume while working on it. This device must not be currently in use.
- **WORK_DIR** - Mount point for the WORK_DEVICE. It will be created
- **CUSTOM_SCRIPT** - Custom script to call within chroot'ed instance volume

## Installation

### Requirements
- Create new Ubuntu work instance or use the existing **non-production** instance.
- Setup AWS custom policy.
- Install awscli and jq.

### Download the files
``````
git clone https://github.com/dzaninov/aws2luks.git
``````

### Configure
``````
vi aws2luks.conf    # review the configuration
vi aws2luks.custom  # review the custom script
``````

### Run
Create the encrypted instance and start it.
``````
 export LUKS_PASSWORD=unlock_password
 export AWS_ACCESS_KEY_ID=iam_access_key_id
 export AWS_SECRET_ACCESS_KEY=iam_secret_access_key
./aws2luks
``````
Space before export is required for bash not to save the command in history.

Instead of exporting AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY stored credentials can be used.

If KEEP_IMAGE is set AMI will stay behind so you can create more instances from it.

### Booting
SSH to instance as **root** to unlock the LUKS device.
``````
echo -n "unlock_password" > /lib/cryptsetup/passfifo
``````
System will boot if the password is correct.

