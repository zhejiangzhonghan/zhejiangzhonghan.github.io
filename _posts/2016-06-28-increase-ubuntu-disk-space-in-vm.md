---
layout: post
title: "Increase Ubuntu Disk Space in VM"
date: 2016-06-28 14:06:31
comments: true
description: "Increase Ubuntu Disk Space in VM"
keywords: "ubuntu, disk, space, modify, increase"
categories:
- tutorial

tags:
- linux

---

I hit an issue that sometimes, the disk size of a VM is not large enough after a few weeks development. I thought it is trivial to increase the disk size since I am using a VM. But it turns out that modify the system disk is not easy. 

I searched online for a while and here is what finally works for me:

Details are explained [here](http://blog.chapus.net/ubuntu-server-increase-disk-space)

1. shutdown VM, increase disk size (please take a snapshot in case anything goes wrong)
2. run cfdisk
    1. select free space
    1. click 'New'
    1. select 'Logical'
    1. select Type => 8e
    1. select Write and confirm by "yes"
    1. select Quit after writing
    1. reboot
3. sudo pvcreate /dev/sda6
4. sudo vgextend ubnt1204 /dev/sda6 (please look up the vgname by vgdisplay)
5. sudo lvextend -l +100%FREE /dev/ubnt1204/root (look up the logiacal volume name by lvdisplay)
6. sudo resize2fs /dev/ubnt1204/root
7. run df -h to confirm new storage size

That's it. Hope it helps. 
