---
layout: post
title: "Logical Volume Manager (LVM) Rehberi"
date: 2013-08-25 16:38
comments: true
categories: [Teknik]
---

LVM yani Logical Volume Manager var olan fiziksel diskleri, insanlık tarihinden bu yana gelen "böl, parçala, yönet"
mantığıyla sanallaştırıp yönetmek amaçlı yazılmış çok yararlı bir program. Bir ya da birden fazla olan disklerinizi
tek bir disk havuzuymuş gibi gösterip, partitions ile karmaşık disk alanı hesaplamalarını sizin yerinize soyutlayıp
yapıyor.

Bir çok linux dağıtımı artık öntanımlı LVM disk formatı seçeneğiyle gelmekte. Özellikle son sürüm işletim sistemleri
aksini belirtmediğiniz takdirde bu şekilde sisteminize kuruluyor.

Bu blog yazısında lvm ile volume grup oluşturma, birden fazla olan disklerinizi volume gruplar
içerisinde toplayıp daha sonrasında toplam disk alanınız üzerinden nasıl disk bölümlemesi yapıldığını anlatacağım.

## Hazırlık

Ben bir Ubuntu 12.10 sanal makinesi üzerinde 16GB ve 14GB lık iki diski lvm ile sanallaştırıp toplamda 30GB lık tek bir
sanal disk yaratıp, daha sonra keyfimize göre bu diski sanal olarak bölümleyeceğim.

Aşağıdaki komut ile disklerinizi görüntüleyin.

`$ sudo fdisk -l`

Aşağıdaki gibi bölümler göreceksiniz :

    `Disk /dev/sdb: 17.2 GB, 17179869184 bytes
    255 heads, 63 sectors/track, 2088 cylinders, total 33554432 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x00000000

    Disk /dev/sdb doesn't contain a valid partition table

    Disk /dev/sdc: 15.0 GB, 15032385536 bytes
    255 heads, 63 sectors/track, 1827 cylinders, total 29360128 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x00000000

    Disk /dev/sdc doesn't contain a valid partition table`

Bu çıktıdan anladığımız, işletim sistemimiz disklerimizden fiziksel olarak haberdar, ancak
hiç bir disk bölümlemesi yapılmadığından dolayı ağlıyor.

Disk bölümlemelerini aşağıdaki gibi yapın :

`$ sudo fdisk /dev/sdb`

Açılan konsolda `m` harfine basarak aşağıdaki gibi yardımcı komutu görüntüleyebilirsiniz :

    Command action
        a   toggle a bootable flag
        b   edit bsd disklabel
        c   toggle the dos compatibility flag
        d   delete a partition
        l   list known partition types
        m   print this menu
        n   add a new partition
        o   create a new empty DOS partition table
        p   print the partition table
        q   quit without saving changes
        s   create a new empty Sun disklabel
        t   change a partition's system id
        u   change display/entry units
        v   verify the partition table
        w   write table to disk and exit
        x   extra functionality (experts only)

`n` harfiyle `/dev/sdb` diskinde bir bölüm oluşturma işine başlayın. Size bu bölümün primary mi logical mı olacağını soracak.
Öntanımlı primary olarak seçim geliyor. Primary seçtikten sonra oluşturduğumuz disk bölümü için 1-4 arası bir değer girmenizi
sizden isteyecek. İsteğinize göre seçip disk bölümünün diskin ne kadarını bölüm için ayarlayacağınızı size soracak. Ben bu örnekte
tüm diski tek bir bölüm olarak oluşturacağım. Daha sonra `w` harfiyle değişiklikleri yapıp fdisk programından çıkış yapacağım.

Yukarıda anlatmaya çalıştığım süreç aşağıdaki gibi gözükecek :

    Command (m for help): n
    Partition type:
        p   primary (0 primary, 0 extended, 4 free)
        e   extended
        Select (default p): 
        Using default response p
        Partition number (1-4, default 1): 
        Using default value 1
        First sector (2048-33554431, default 2048): 
        Using default value 2048
        last sector, +sectors or +size{K,M,G} (2048-33554431, default 33554431): 
        Using default value 33554431

        Command (m for help): w
        The partition table has been altered!

        Calling ioctl() to re-read partition table.
        Syncing disks.

    vagrant@precise32:~$

Aynı işlemi `/dev/sdc` için de yapın.

<!--more-->

## LVM Rehberi

Şu anda sistemimizdeki 16GB ve 14GB lık iki diskimizi oluşturduk. Sıra bu diskleri `lvm` ile sanallaştırmaya geldi.

`lvm2` programını aşağıdaki şekilde indirin :

`$ sudo apt-get install lvm2`

Eklediğimiz diskleri physical volume olarak oluşturmamız gerekecek :

`$ sudo pvcreate /dev/sdb1 /dev/sdc1`

Physical volume disklerinizi listeleyin :

`$ sudo pvs`

    vagrant@precise32:~$ sudo pvs
        PV         VG        Fmt  Attr PSize  PFree 
        /dev/sda5  precise32 lvm2 a-   79.76g     0 
        /dev/sdb1            lvm2 a-   16.00g 16.00g
        /dev/sdc1            lvm2 a-   14.00g 14.00g

Detaylı liste için :

`$ sudo pvdisplay`

Physical Volume'lerinizin herhangi bir Volume Grubuna ait olmadığı yukarıda gözüküyor. Sıra geldi bu iki diskimizi
`vg1` isimli bir volume grup içine alıp, iki ayrı fiziksel diskimizi tek bir diskmiş gibi sanallaştıralım :

`$ sudo vgcreate vg1 /dev/sdb1 /dev/sdc1`

Volume Group listelemek için :

`$ sudo vgdisplay`

Çıktısını aşağıdaki gibi :

    vagrant@precise32:~$ sudo vgdisplay 
    --- Volume group ---
    VG Name               vg1
      System ID             
        Format                lvm2
        Metadata Areas        2
        Metadata Sequence No  1
        VG Access             read/write
        VG Status             resizable
        MAX LV                0
        Cur LV                0
        Open LV               0
        Max PV                0
        Cur PV                2
        Act PV                2
        VG Size               29.99 GiB
        PE Size               4.00 MiB
        Total PE              7678
        Alloc PE / Size       0 / 0   
        Free  PE / Size       7678 / 29.99 GiB
        VG UUID               K9QzS7-D8aI-O24O-Rcsb-2aMd-pY2D-3zjP5w

Yukarıda görebileceğiniz gibi 2 metadata alanı (2 disk) ve toplamda 29.99 GiB tek bir sanal disk oluşturduk.

Şimdi sıra oluşturduğumuz vg1 isimli Volume Group'u istediğimiz büyüklükte istediğimiz kadar sanal diske bölüp
sistemimizde kullanılabilir hale getirmeye geldi.

Ben örnek olarak 30 GiB lık toplam boş alanı olan diskimi `lv01` ve `lv02` 15 ve 14er GiB lık sanal disklere böleceğim.

`NOT:` Tembel olduğumdan dolayı tam 30 GiB alan hesaplamaya üşendim açıkcası. :)

`$ sudo lvcreate -L +15G -n lv01 vg1`

`sudo lvcreate -L +14G -n lv02 vg1`

Şimdi oluşturduğumuz Logical Volume'larımızı listeleyelim :

`$ sudo lvdisplay`

    vagrant@precise32:~$ sudo lvdisplay 
    --- Logical volume ---
    LV Name                /dev/vg1/lv01
    VG Name                vg1
    LV UUID                W9TxNd-ORun-k0Cx-2psu-iI1p-8fQc-SroVSm
    LV Write Access        read/write
    LV Status              available
    # open                 0
    LV Size                15.00 GiB
    Current LE             3840
    Segments               1
    Allocation             inherit
    Read ahead sectors     auto
    - currently set to     256
    Block device           252:2
    
    --- Logical volume ---
    LV Name                /dev/vg1/lv02
    VG Name                vg1
    LV UUID                zP1NJX-QwlD-JzBm-PYLU-5nFZ-i9vq-q7n3Vv
    LV Write Access        read/write
    LV Status              available
    # open                 0
    LV Size                14.00 GiB
    Current LE             3584
    Segments               2
    Allocation             inherit
    Read ahead sectors     auto
    - currently set to     256
    Block device           252:3

Logical Volume'larımızı da oluşturduk. Sıra geldi bunu sistemimizde kullanılabilir hale getirmeye :

Ubuntu ortamında geliştirme yaptığım için `mkfs` programı kullanarak oluşturduğum sanal diskleri `ext4` ile formatlayacağım :

`$ sudo mkfs.ext4 /dev/mapper/vg1-lv02 `

Çıktısı aşağıdaki gibi olacak :

    vagrant@precise32:~$ sudo mkfs.ext4 /dev/vg1/lv01
    mke2fs 1.42 (29-Nov-2011)
    Filesystem label=
    OS type: Linux
    Block size=4096 (log=2)
    Fragment size=4096 (log=2)
    Stride=0 blocks, Stripe width=0 blocks
    983040 inodes, 3932160 blocks
    196608 blocks (5.00%) reserved for the super user
    First data block=0
    Maximum filesystem blocks=4026531840
    120 block groups
    32768 blocks per group, 32768 fragments per group
    8192 inodes per group
    Superblock backups stored on blocks: 
    32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208

    Allocating group tables: done
    Writing inode tables: done
    Creating journal (32768 blocks): done
    Writing superblocks and filesystem accounting information: done 

Aynı işlemi oluşturduğumuz diğer Logical Volume için de yapıp diski formatlayın.

Artık `ext4` olarak formatladığımız 15 GB'lık diskimizi sistemde kullanabiliriz.

Ben `/mnt` dizini altında disk1 isimli bir dizin oluşturup, bu dizin altına `lv01` diskimi `mount` edeceğim.

`$ sudo mkdir /mnt/disk1`

`$ sudo mount /dev/vg1/lv01 /mnt/disk1`

Ve diskiniz hazır, diğer hazırladığınız diski de dilerseniz aynı yöntemle formatlayıp sisteme mount edebilirsiniz.

Diski görebilmek için :

`$ df -h`

    vagrant@precise32:/mnt/disk2$ df -h
    Filesystem                  Size  Used Avail Use% Mounted on
    /dev/mapper/precise32-root   79G  2.1G   73G   3% /
    udev                        178M  4.0K  178M   1% /dev
    tmpfs                        74M  296K   74M   1% /run
    none                        5.0M     0  5.0M   0% /run/lock
    none                        185M     0  185M   0% /run/shm
    /dev/sda1                   228M   24M  192M  12% /boot
    /vagrant                    290G   65G  225G  23% /vagrant
    /dev/mapper/vg1-lv01         15G  354M   14G   3% /mnt/lv01
    /dev/mapper/vg1-lv02         14G  340M   13G   3% /mnt/disk2

Hepsi bu kadar :)
