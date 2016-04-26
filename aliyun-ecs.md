# aliyun ecs manage

## mount data disk
```
fdisk -l
fdisk /dev/xvdb
mkfs.ext3 /dev/xvdb1
mount /dev/xvdb1 /data
```

## auto mount on boot
```
vi /etc/fstab
/dev/xvdb1 /data ext3 defaults 0 0
```
