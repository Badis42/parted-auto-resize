## WAT

in short `resize a partition non-interactive to its maximum size`

#### The long story..

Since using fdisk in this case is pretty complicated due to the case that non-interactive ways are probably not possible or very complicated using printf, i want to used parted resizepart for resizing a partition to its maximum size.

This can be used in scenarios like disk-resized ( hypervisor / virtualization ) and now you need to adjust your logical volume / pv to the new size (LVM case) or you want to adjust the partition size of a normal partition.

So lets assume i want to resize partition /dev/sda1 on disk /dev/sda to its maximum possible size - how would i do this without getting asked any questions at all.

Eventhough `parted /dev/sda resizepart 1` exists, it needs **me to calculate** and enter the maximum disk size - so how to automate this would be the next question - and the answer was the reason `parted auto resize` has been written by me.

## usage 
Save the script above as `resize.sh` and make it executable
    
    # resize the fourth partition to the maximum size, so /dev/sda4
    # this is no the sandbox mode, so no changes are done
    ./resize.sh /dev/sda 4

    # apply those changes
    ./resize.sh /dev/sda 4 apply

## scenarios / motivation

Since debian does not allow your to preseed a LVM configuration, which uses a disk as a pv / vg, but always a partition, you need a convinient way to resize the pv partition, so you can resize any logical volumes. It can be done using fdisk, but this needs ot be done by hand, interactive with a lot more steps ( usual delete / create / set type to Linux LVM / write / reload partition table 

Assuming your pv / vg is created on /dev/sdb1 named vgdata and the LV to resize is named data and you resized your disk using your hypervisor.

Now using this script, all you do is

    ./resize.sh /dev/sdb 1 apply
    pvresize /dev/sdb1
    lvextend -r /dev/mapper/vgdata-data -l 100%FREE

thats it! Note, if you happen to notice -r in lvextend, yes thats pretty cool, it calls `resize2fs /dev/mapper/vgdata-data` for us automatically after the partition resize ( to adjust the filesystem size )
