#Learn btrfs


--create a btrfs volume
sudo mkfs.btrfs -L MyDataRaid1 /dev/sdb

-- mount the new created btrfs device 
sudo mount /dev/sdb /media/lusu/MyDataRaid1/

-- check disk space ohe btrfs device
sudo btrfs fi df -h /media/lusu/MyDataRaid1/
sudo btrfs fi show

-- add a second device
btrfs device add /dev/sdc /mnt/aem_raid1/
sudo btrfs fi df -h /media/lusu/MyDataRaid1/
sudo btrfs fi show

-- convert to RAID1
sudo btrfs balance start -dconvert=raid1 -mconvert=raid1 /media/lusu/MyDataRaid1/
sudo btrfs fi df -h /media/lusu/MyDataRaid1/
sudo btrfs fi show

-- add more devices (if needed). notice that raid type remains unchaged (RAID1)
sudo btrfs device add /dev/sdd /media/lusu/MyDataRaid1/
sudo btrfs balance /media/lusu/MyDataRaid1/
sudo btrfs fi df -h /media/lusu/MyDataRaid1/
sudo btrfs fi show

-- check DIsk UUID
blkid /dev/sdX

-- mount my Raid in boot
--- add this line to /etc/fstab
UUID=39604107-47f9-4c1e-80a0-e11f72fd8048 /media/lusu/MyDataRaid1 btrfs defaults 0 0

-- check disk space
btrfs fi df /<mounted_diskname>  -- ex: btrfs fi df /media/lusu/MyDataRaid1/

-- example :
lusu@deb8:~$ sudo btrfs fi show
Label: 'MyDataRaid1'  uuid: 39604107-47f9-4c1e-80a0-e11f72fd8048
	Total devices 4 FS bytes used 147.52MiB
	devid    3 size 8.00GiB used 32.00MiB path /dev/sdd
	devid    4 size 8.00GiB used 1.25GiB path /dev/sde
	devid    5 size 8.00GiB used 32.00MiB path /dev/sdf
	devid    6 size 8.00GiB used 1.25GiB path /dev/sdc

Btrfs v3.17
lusu@deb8:~$ sudo btrfs fi df -h /media/lusu/MyDataRaid1/
Data, RAID1: total=1.00GiB, used=147.23MiB
System, RAID1: total=32.00MiB, used=16.00KiB
Metadata, RAID1: total=256.00MiB, used=272.00KiB
GlobalReserve, single: total=16.00MiB, used=0.00B

-- repalce a disk from a RAID1 volume. 
--- normally you could not remove a disk if it's in raid1. you got the error below: 
--- ERROR: error removing the device '/dev/sdi' - unable to go below two devices on raid1
--- to do so, here are the steps to go:
---- 1. first you have to convert your existing Btrfs volume to "single":
sudo btrfs balance start -dconvert=single -mconvert=single -dconvert=single /mnt/MyPersonalData/

---- 2. then remove a disk from your RAID1 by running the command line:  
sudo btrfs device delete /dev/sdi /mnt/MyPersonalData/ -- /mnt/MyPersonalData = is the name of the mounted volume 

---- 3. make sure now your btrfs volume has only one disk. 
sudo btrfs fi show MyBTRFS
Label: 'MyBTRFS'  uuid: c8f7b7a8-0425-472f-a3be-c66cb7c7e4da
	Total devices 1 FS bytes used 98.37MiB
	devid    2 size 3.00GiB used 800.00MiB path /dev/sdh

---- 4. make sure you volume is mounted correctely in /etc/fstab:
nano /etc/fstab
..
UUID=c8f7b7a8-0425-472f-a3be-c66cb7c7e4da /mnt/MyPersonalData btrfs defaults 0 0
..
 
---- 5. reboot your system
sudo reboot

---- 6. check if your btrfs volume is now in "single" mode, and have only one disk  
sudo btrfs fi df /mnt/MyPersonalData/
Data, single: total=512.00MiB, used=98.26MiB
System, single: total=32.00MiB, used=16.00KiB
Metadata, single: total=256.00MiB, used=224.00KiB
GlobalReserve, single: total=16.00MiB, used=0.00B

---- 7. add the new disk to your btrfs volume 
sudo btrfs device add /dev/sdi /mnt/MyPersonalData/


---- 8. convert back your btrfs volume to raid1 mode
sudo btrfs balance start -dconvert=raid1 -mconvert=raid1 -dconvert=raid1 /mnt/MyPersonalData/

---- 9. check again if your btrfs volume is now of 2 disks 
sudo btrfs fi show MyBTRFS
Label: 'MyBTRFS'  uuid: c8f7b7a8-0425-472f-a3be-c66cb7c7e4da
	Total devices 2 FS bytes used 98.37MiB
	devid    2 size 3.00GiB used 800.00MiB path /dev/sdh
	devid    3 size 2.00GiB used 800.00MiB path /dev/sdi

---- 10. done :)
