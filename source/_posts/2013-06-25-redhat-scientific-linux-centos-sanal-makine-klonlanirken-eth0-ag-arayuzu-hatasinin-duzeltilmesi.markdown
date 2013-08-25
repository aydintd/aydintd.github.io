---
layout: post
title: "RedHat Scientific Linux CentOS Sanal Makine Klonlarının eth0 Ağ Arayüzü Hatasının Düzeltilmesi"
date: 2013-06-25 02:25
comments: true
categories: [Teknik]
---

Scientific Linux 6 sürümünü VirtualBox'da "bare" olarak klonlayıp, ağ konfigürasyonunu
yaparken ilginç bir hatayla karşılaştım. İnternette bir çok yerde SL, CentOS ve
türev sürümlerinin VirtualBox klonlarının internet erişimini sağlayamadığından yakınan
baya bir kullanıcı girdisi okudum.  
Bir kaç okuma yaptıktan sonra, şöyle bir şey keşfettim :  

VirtualBox, VMWare gibi sanallaştırma araçları, ilgili klonların her birine
kendisi bir MAC adresi atıyor. Ancak SL, CentOS gibi sürümlerin kernel'leri kendilerine atanan
bu yeni konfigürasyonu ilk başta update edemiyorlar. Eski kernel'da kayıtlı MAC adresiyle
VBox'un klonlara atadığı MAC adresi de örtüşmediğinden sanal makineniz sadece loopback
arayüzünü başlatabiliyor ve eth0 ölü halde kalıyor. Dolayısıyla klonlar internete
çıkamıyorlar. 

##  Peki Çözüm Nasıl Olmalı?

*   Kernel'ın başta oluşturduğu "ağ arayüzü kuralları" dosyasını silelim. Böylece
klon yeniden başlatıldığında kernel kendine yeni bir kural dosyası üretebilsin
ve böylece VM tarafından atanan yeni MAC adresini kendine kaydedebilsin :

`$ sudo rm -rf /etc/udev/rules.d/70-persistent-net.rules`

`$ sudo reboot`

*   Makine yeniden başladıktan sonra sıra geldi ağ yapılandırmasına.
SL 6.4 de default olarak gelen text editörü `vi` olduğundan dosya
düzenlemelerini vi ile göstereceğim :

`$ sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0`

*   Dosyada aşağıdaki değişiklikleri uygulayalım :

`MACADDR` girdisini kaldıralım.

`UUID` girdisini kaldıralım.

ve dosyayı kaydedip çıkalım.

*   Ağ konfigürasyonumuzu tamamladık, servisi yeniden başlatmalıyız ki girdiğimiz
ayarlar klon tarafından görülebilsin.

`$ service network restart`

*   ve İnternete çıkış yaptık mı? Ping'leyerek kontrol edelim :

`$ ping 8.8.8.8`

Ta-daa! Klonunuz internete çıkmaya hazır. Tek yapmanız gereken ihtiyacınız olan servisi
"yum"lamak.

`Önemli Not:` VirtualBox'da Ağ Seçeneği NAT olarak ayarlı olmalı.

##   Kaynakça

`http://blog.roozbehk.com/post/35156862882/redhat-add-network-eth`

`http://www.envision-systems.com.au/blog/2012/09/21/fix-eth0-network-interface-when-cloning-redhat-centos-or-scientific-virtual-machines-using-oracle-virtualbox-or-vmware/`
