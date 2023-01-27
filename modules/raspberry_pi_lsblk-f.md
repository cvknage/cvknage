```console
lsblk -f
```
[output]
```
pi@raspberrypi:~ $ lsblk -f
NAME        FSTYPE  FSVER LABEL       UUID                                 FSAVAIL FSUSE% MOUNTPOINT
sda                                                                                       
├─sda1      vfat    FAT32 EFI         D2D7-4855                                           
├─sda2      hfsplus       Monterey    99b629a1-2994-45cf-b8c1-cff6d7450694                
├─sda3      hfsplus       Ventura     3516e24d-84c2-4f9e-9559-71de730a7277                
├─sda4      ext4    1.0   TimeMachine fc924351-26c8-4c01-a25f-e44a6869c5f1  614.6G    25% /media/pi/TimeMachine
└─sda5      ext4    1.0   NAS         665507c3-aee6-41bc-8be8-3594a65468d9    1.3T    45% /media/pi/NAS
mmcblk0                                                                                   
├─mmcblk0p1 vfat    FAT32 boot        444F-BE04                             200.3M    21% /boot
└─mmcblk0p2 ext4    1.0   rootfs      665507c3-aee6-41bc-8be8-3594a65468d9   19.4G    28% /
pi@raspberrypi:~ $ 
```