# hubic

This is the setup process for using rclone which sets up an encrypted rsync with your cloud storage provider. While you do not have to be using hubiC[1] to follow this guide it was definitely tailored towards it.

hubiC is a cloud storage service provided by OVH. I use it to backup all of my data, whether it be from my house or from my dedicated server (also hosted by OVH).

## Install rclone
First install rclone.

> yay -S rclone

## Authorize with Cloud Provider
Run the rclone config.

> rclone config

Create a new remote by selecting n and pressing <ENTER>.

Specify a name for the remote for use in commands. For this I am using hubiC.

Select which cloud provider you are using, again I am selecting 8 for hubiC.

Leave the Client Id and Client Secret blank for hubiC.

Select YES to auto config (provided Xorg is running and a web browser is available). Then follow the instructions and login to your cloud provider so that rclone can get the proper authentication.

Next accept the key download by selecting YES the token is correct.

The remote should now be properly configured.

## Encryption

Before configuration of the encrypted remote is setup, make sure you have a folder created on your cloud provider and setup properly for your needs before you continue. I am going to be setting up a folder for an SD Card that permanently sits in my laptop, the folder I created is called sdcard.

Once the directory is created you can run rclone again.

> rclone config

Again we want to setup a N new remote.

This time select option 5 for Encrypt/Decrypt a remote.

Next specify which remote and directory you would like to encrypt/decrypt, this has the format remote:directory.

#### For hubiC in particular if you can see the folder in the web interface you must prefix it with default, normally it would just be hubiC:/sdcard/
 
hubiC:default/sdcard/

Type standard for encrypted file names.

Select Y to set a password for the encrypted container.

Then repeat this process for a salt if you want added security or chose a non-random password.

```
WARNING: DO NOT forget the password or salt to the container or all of the data inside will be irretrievable!
```

Finally confirm with Y to create the encrypted remote.

## Sync
Integrity can be checked to see the difference.

> rclone check /mnt/sdcard sdcard:

To put data to the cloud one has two options: copy and sync.

Copy will copy new files from the local computer to the cloud, but it will not remove files from the cloud that have been removed locally. This is particularly good for adding things to an existing container, such as a backup folder.

Sync will mirror the local computer to the cloud, this will delete files from the cloud that have been removed locally. This can be particularly useful for application database folders, or any data that needs to be 1:1 synchronized.

If you need to have a running copy of what is on the cloud I would highly recommend mounting your container from the cloud instead.

```
# rclone copy /mnt/sdcard sdcard:
# rclone sync /mnt/sdcard sdcard:
```

To restore a folder from the cloud to the local computer simply use the sync option.

## Mounting
One can also mount their folder direct from the cloud, this can be useful for media servers.

> rclone mount --umask 0 --allow-other --max-read-ahead 200M sdcard: /mnt/sdcard &

If you then need to unmount it for any reason you can with the following.

> fusermount -u /mnt/sdcard

Another option is to create a systemd user service.

```
[Unit]
Description=rclone FUSE mount
After=network.target

[Service]
Type=simple
User=USER
ExecStart=/usr/sbin/rclone mount --umask 0 --allow-other --max-read-ahead 200M sdcard: /mnt/sdcard
ExecStop=/bin/fusermount -u /mnt/sdcard
Restart=always

[Install]
WantedBy=multi-user.target
```

### References
[^ rclone](https://rclone.org/overview/). Overview of cloud storage systems

### Source
[^ Encrypted Cloud Storage](https://kyau.net/wiki/ArchLinux:rclone)
