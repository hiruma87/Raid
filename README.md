# Raid
## Windows
- Open PowerShell in Administrator
- Search for available Disk for Pooling by typing
  ```bash
  Get-PhysicalDisk –CanPool $True
  ```
- If want to use all the Disk, type
  ```bash
  $disks = Get-PhysicalDisk `
    -CanPool $true
  ```
  ```bash
  New-StoragePool `
    -FriendlyName "*Your Pool Name*" `
    -StorageSubSystemFriendlyName "*Your Space Name*" `
    -PhysicalDisks $disks
  ```
  or specify which Disk to use
  ```bash
  -PhysicalDisks 1,2,4
  ```
- Then we create the the Storage
  ```bash
  New-VirtualDisk `
  -StoragePoolFriendlyName *Your Pool Name* `
  -ProvisioningType Thin `
  -Interleave 128KB (see side notes) `
  -FriendlyName *Your Space Name* `
  -Size 1.4TB `
  -ResiliencySettingName Simple `
  -NumberOfColumns 3
  ```
  #### Side notes (it improve read/write speed)
  ```
  (Number of Columns-1)*(Interleave Size). 
  Since the storage spaces interleave sizes have the following possible values: (4KB, 8KB, 16KB, 32KB, 64KB, 128KB, 256KB and so on), and common NTFS cluster sizes are (4KB, 8KB, 16KB, 32KB, 64KB), you can see that the following combinations will yield best alignment.
  3 columns, 32KB interleave => 64KB data stripe size, matches 64KB NTFS cluster size.
  5 columns, 16KB interleave => 64KB data stripe size, matches 64KB NTFS cluster size.
  ```
- Format the storage space according to above mentioned in side notes

## Linux
- Install `mdadm`
- Create Raid0 (if you want to change the raid level, change the level, raid-devices for disk count)
  ```bash
  mdadm --create --verbose --level=0 --metadata=1.2 --chunk=256 --raid-devices=3 /dev/md/MyRAID5Array /dev/sdb1 /dev/sdc1 /dev/sdd1
  ```
#### Formating the RAID
Example 1. RAID0
Example formatting to ext4 with the correct stripe width and stride:
- Hypothetical RAID0 array is composed of 2 physical disks.
- Chunk size is 512 KiB.
- Block size is 4 KiB.
stride = chunk size / block size. In this example, the math is 512/4 so the stride = 128.
stripe width = # of physical data disks * stride. In this example, the math is 2*128 so the stripe width = 256.
```
mkfs.ext4 -v -L myarray -b 4096 -E stride=128,stripe-width=256 /dev/md0
```
___
Example 2. RAID5
Example formatting to ext4 with the correct stripe width and stride:
- Hypothetical RAID5 array is composed of 4 physical disks; 3 data discs and 1 parity disc.
- Chunk size is 512 KiB.
- Block size is 4 KiB.
stride = chunk size / block size. In this example, the math is 512/4 so the stride = 128.
stripe width = # of physical data disks * stride. In this example, the math is 3*128 so the stripe width = 384.
```
mkfs.ext4 -v -L myarray -b 4096 -E stride=128,stripe-width=384 /dev/md0
```
___
Example 3. RAID10,far2
Example formatting to ext4 with the correct stripe width and stride:
- Hypothetical RAID10 array is composed of 2 physical disks. Because of the properties of RAID10 in far2 layout, both count as data disks.
- Chunk size is 512 KiB.
- Block size is 4 KiB.
stride = chunk size / block size. In this example, the math is 512/4 so the stride = 128.
stripe width = # of physical data disks * stride. In this example, the math is 2*128 so the stripe width = 256.
```
mkfs.ext4 -v -L myarray -b 4096 -E stride=128,stripe-width=256 /dev/md0
```
___
- Let's say you already had existing mdadm raid config, you can try re-assemble the raid so it can be detected if you fresh install or move the raid to another PC
  ```bash
  mdadm --assemble --scan
  ```
