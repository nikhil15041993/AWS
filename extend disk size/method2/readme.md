List the Attached Devices:

    You can list the attached devices on your EC2 instance using the lsblk command. Run the following command to view the list:



```
lsblk
```
Format the New Volume:

    If the new volume is not already formatted, you'll need to format it with a filesystem of your choice. For example, to format it with ext4:


```
sudo mkfs -t ext4 /dev/xvdf
```

Create a Mount Point:

    Create a directory where you want to mount the new volume. For example:

```
sudo mkdir /mnt/mynewvolume
```

Mount the Volume:

    Mount the EBS volume to the directory you just created:

```
sudo mount /dev/xvdf /mnt/mynewvolume
```

Configure Auto-Mount (Optional):

    If you want the volume to be mounted automatically at boot, you can add an entry to the /etc/fstab file. Open the file in a text editor:

```
sudo nano /etc/fstab
```

    Add a line at the end of the file like this:

```
/dev/xvdf /mnt/mynewvolume ext4 defaults 0 0
```
    Save and exit the text editor.

Verify the Mount:

    You can verify that the volume is successfully mounted by listing the contents of the mount point:

```
ls /mnt/mynewvolume
```
