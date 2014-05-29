---
layout: post
title: "LTSP-DHCP Konfigurasyonu"
date: 2012-05-10 18:42
comments: true
categories: [Kişisel]
---

##  LTSP Kurulumu

- LTSP sunucuyu kur

  `$ sudo apt-get install ltsp-server-standalone`

  Bu komut LTSP ve (eğer kurulu değilse) DHCP sunucuyu birlikte kurar.

## LTSP Yapılandırması

- `/etc/ltsp/ltsp-build-client.conf` dosyasını düzenle
        
        `# The chroot architecture.
         ARCH=i386

         # ubuntu-desktop and edubuntu-desktop are tested.
         # If you test with [k|x]ubuntu-desktop, edit this page and mention if it worked OK.
         # kubuntu lucid (10.10) working okay.
         FAT_CLIENT_DESKTOPS="ubuntu-desktop"

         # Space separated list of programs to install.
         # The java plugin installation contained in ubuntu-restricted-extras
         # needs some special care, so let's use it as an example.
         LATE_PACKAGES="
             ubuntu-restricted-extras
             gimp
             nfs-client"

         # This is needed to answer "yes" to the Java EULA.
         # We'll create that file in the next step.
         DEBCONF_SEEDS="/etc/ltsp/debconf.seeds"

         # This uses the server apt cache to speed up downloading.
         # This locks the servers dpkg, so you can't use apt on
         # the server while building the chroot.
         MOUNT_PACKAGE_DIR="/var/cache/apt/archives/"`

## DHCP Yapılandırılması

- `/etc/ltsp/dhcpd.conf` dosyasını düzenle
          
        authoritative;
        allow bootp;

        subnet 10.0.2.0 netmask 255.255.255.0 {
            range 10.0.2.20 10.0.2.250;

            option domain-name "example.com";
            option domain-name-servers 192.168.111.5;
            option broadcast-address 10.0.2.255;
            option routers 10.0.2.2;
        #    next-server 192.168.0.1;

        #    get-lease-hostnames true;
            option subnet-mask 255.255.255.0;
            option root-path "/opt/ltsp/i386";
            if substring( option vendor-class-identifier, 0, 9 ) = "PXEClient" {
                filename "/ltsp/i386/pxelinux.0";

            } else {
                filename "/ltsp/i386/nbi.img";
            }
        }
  

  `option domain-name-servers` karşısına yazılan adres, internetinizin o anki
  size verdiği DNS adresi olmalı. Aksi takdirde daha sonradan kurulacak olan
  istemcinin internete bağlı gözüktüğünü ancak herhangi bir web sitesini
  açamadığını göreceksiniz.

- `/etc/default/isc-dhcp-server` dosyasında arayüzü (`INTERFACES`) düzenle
  `INTERFACES`'i etkinleştir

  ```
  INTERFACES = "eth1"
  ```

- Sunucuları yeniden başlat

  `$ service networking restart`

  `$ service dhcp3-server restart`

## İstemci Kurulumu

- İstemci paketini kur

  `$ ltsp-build-client --fat-client --fat-client-desktop ubuntu.desktop --arch i386 --skipimage`

  Bu komut istemcileri sunucu içerisinde `/opt/ltsp/i386` dizini altında
  oluşturur.  İstemciye hangi dağıtımı kurmak istiyorsanız, kodda o işletim
  sistemini yazmalısınız; ör. Ubuntu için `ubuntu.desktop`

<!--more-->

## İstemci Güncelle

- İstemciye gir

  `$ chroot /opt/ltsp/i386`

- İstemci imajını güncelle

  `$ ltsp-update-sshkeys`

  `$ ltsp-update-image --arch i386`

## Ağ Yapılandırması

- `/etc/network/interfaces` dosyasını düzenle

        auto lo

        iface lo inet loopback

        auto eth1

        iface eth1 inet static
               address 10.0.0.2
               netmask 255.255.255.254
               broadcast 10.0.2.255

        auto eth2

        iface eth2 inet dhcp

  Tüm bunlardan sonra, sunucunun internet bağlantısının olduğunu ancak,
  istemcinin internet bağlantısının olmadığını göreceksiniz.

  Bunun nedeni istemcinin sunucudan internete bağlanma isteği göndermesine
  karşın sunucunun buna izin vermemesidir. Yani sunucu, kendi üzerinden
  istemcilere internete çıkma hakkı tanımıyor.

  İç ağdaki makinaların internet üzerindeki makinalarla haberleşmesini
  sağlayabilmek için NAT Maskelemesi (Network Access Translation Masquerade)
  yapılmalıdır.

## NAT

- ` /etc/init.d/rc.local` dosyasını düzenle

  `$ iptables -t nat -A POSTROUTING -o eth2 -j MASQUERADE`

  `$ echo "1" > /proc/sys/net/ipv4/ip_forward`

