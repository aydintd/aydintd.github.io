---
layout: post
date: 2012-09-10 15:27
comments: true
categories: [Teknik]
title: ".csvden Kullanıcı/Grup Ekleme"
---

## Kullanıcı/Grupların Eklenmesi
 
- `$ sudo ./lab-kullanıcıları-ekle Dosya_ADI` komutunu çalıştır

  Öntanımlı kabuğu değiştir
  
        #!/bin/bash
        useradd -Ds /bin/bash
  
        for isim in `cat $1`; do
        useradd -d /home/$isim -p `mkpasswd 123456` $isim
          mkdir /home/$isim
          chown -R $isim:$isim /home/$isim
        done

  Betiğe argüman olarak verilecek dosya formatı şu şekilde olmalıdır:  

        cankurnaz  
        aydindoyak  
  
  UYARI: Bu dosyada türkçe karakter ve noktalama işaretleri olmamalıdır; bu  
  işlemin öncesinde dosyayı belirtilen formata uygun hale getirin.
