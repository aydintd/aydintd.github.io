---
title: "Ubuntuda Makine Adı Değiştirme"
layout: post
date: 2012-10-14 20:12
comments: true
categories: [Teknik]
---

- Ubuntu 12.04 işletim sistemi sürümünde  
makine adı kolayca değiştirilebiliyor.  
Bu işlemi sadece iki adımda yapabilirsiniz :  

- `$ gksudo gedit /etc/hostname` içine girin  
ve açılan dosyaya olmasını istediğiniz makine ismini  
yazın.  

- Daha sonra; `$ gksudo gedit /etc/hosts` komutunu  
çalıştırdığınızda aşağıdaki gibi bir dosya görünümü  
görüyor olmanız lazım :

        127.0.0.1      localhost  
        127.0.1.1      peekaboo
        
        # The following lines are desirable for IPv6 capable hosts
        ::1 ip6-localhost ip6-loopback
        fe00::0 ip6-localnet
        ff00::0 ip6-mcastprefix
        ff02::1 ip6-allnodes
        ff02::2 ip6-allrouters
        
  `127.0.1.1      peekaboo` satırında peekaboo yazan kısmı bir  
  önceki adımda olmasını istediğiniz makine ismiyle değiştirin.  
  
- Son olarak `$ sudo reboot` diyerek sisteminizi yeniden  
başlatın. Başlangıç sayfasında ve uçbiriminizde makine adınızın  
değiştiğini göreceksiniz.  

- Dip not : Örnekteki peekaboo kendi kullandığım makine ismidir.
