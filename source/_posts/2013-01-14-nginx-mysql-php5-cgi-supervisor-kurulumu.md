---
title: "Nginx-mysql-php5 cgi-supervisor Kurulumu"
layout: post
date: 2013-01-14 1:44
comments: true
categories: [Teknik]
---
  php denemeleri yapabileceğimiz nginx web sunucusu için   
  gereken ortamı sırasıyla kuralım :    

### Kurulum İçerikleri  

  *  mysql Kurulumu  
  *  nginx Kurulumu ve Konfigürasyonu  
  *  php5-cgi Kurulumu  
  *  supervisor Kurulumu  

#### Mysql Kurulumu  

  Paketi aşağıdaki şekilde kuruyoruz :  

  `$ sudo apt-get install mysql-server mysql-client`  

  Kurulumda bizden mysql root password istiyor. Uygun bir  
  password girip kurulumun tamamlanmasını bekliyoruz.  

  Kurulum bittikten sonra mysql doğru çalışıyor mu diye  
  kontrol etmemiz gerek. Mysql prosesi var mı kontrol edelim :    

  `$ sudo ps ax | grep mysqld`  
	  
	  6234 ?            Ssl  O:OO /usr/sbin/mysqld
	  6546 pts/0        S+   0:00  grep --color=auto mysqld

  Yukarıdakine benzer bir çıktı veriyorsa mysql doğru kurulmuş demektir.  
  Ancak dilerseniz mysql e bir giriş yapalım ve emin olalım.  

  `$ mysql -u root -p` 

  Kurulum yaparken ki root şifrenizi girip mysql konsolunuzu görüntüleyin.  
  `exit` diyerek çıkış yapalım.  

<!--more-->

#### Php5 &  php-cgi Kurulumu

  Aşağıdaki komutla paketi kuralım :

  `$ sudo apt-get install php5 php5-cgi`

  Kurulum bittikten sonra php-cgi processi çalışıyor mu kontrol edelim.  

  `$ sudo ps ax | grep cgi`
         
	 4580 pts/0    S+     0:00 grep --color=auto cgi
  
  Kuracağımız nginx sunucusunda php5 çalıştırabilmek için FastCGI kullanacağız.  
  Bu yüzden gerekli modülleri aktivesi için aşağıdaki komutu çalıştırıyoruz :  

  ```
  $ sudo apt-get install php5-mysql php5-curl php5-gd php5-idn php-pear     
  php5-imagick php5-imap php5-common php5-mcrypt php5-memcache    
  php5-mhash php5-ming php5-ps php5-pspell php5-recode php5-snmp     
  php5-sqlite php5-tidy php5-xmlrpc php5-xsl    
  ```      
 
  Bu modülleri kurduktan sonra sitelerimizi oluşturabilmek adına    
  /var/www dizinine gerekli izni veriyoruz.  
  (eğer /www dizini yoksa kendiniz oluşturun[1]) :    
  
  `$ sudo mkdir /var/www` [1]  
  `$ sudo chmod 777 -R /var/www`  

  Denemek için konsolunuzda :  

        cat >/var/www/test.php
	<?php
	phpinfo();
	?>
	ctrl + d

  Dedikten sonra web tarayıcınıza `http://localhost/test.php`  
  yazdığınızda sizi php nin test sayfasının karşılamasını umuyoruz.    
  Eğer bir terslik varsa şu ana kadar ki kurulumlarınızı gözden geçirin.    


#### nginx Kurulumu  

  nginx aşağıdaki komutla kuralım :  

  `$ sudo apt-get install nginx`  

  Kurulum bittikten sonra nginx i tekrar başlatalım :  

  `$ sudo service nginx restart`
        
	Starting nginx: the configuration file /etc/nginx/nginx.conf 
	syntax is ok
        configuration file /etc/nginx/nginx.conf 
	test is successful
	nginx.  

  nginx prosesi çalışıyor mu kontrol edelim : 

  `$ sudo netstat -pantu | grep :80`

        (Not all processes could be identified, non-owned process info
	will not be shown, you would have to be root to see it all.)
	tcp        0      0 0.0.0.0:80              0.0.0.0:*   DİNLE  -  

  Şimdi sıra nginx i konfigüre etmeye geldi.  

  Aşağıdaki ayar dosyasını makinemize indirelim :  

  `$ wget http://aydintd.me/file/nginx-ayar`  

  Ve bu ayar dosyasını nginx in default ayar dosyasına yükleyelim.  

  `$ sudo install -m 644 nginx-ayar /etc/nginx/sites-available/default`  

  Dizin izinlerini ayarlayalım : 

  `$ sudo chown -R www-data:www-data /var/www/`

  Web tarayıcımızdan http://localhost a girdiğimizde  
  `Welcome to nginx` gibi bir başlıkla karşılaşmamız gerek.  

  Makineyi tekrar başlatın.

  `$ sudo reboot`

  nginx i tekrar başlatın.

  `$ sudo service nginx start`

#### Supervisor Kurulumu

  Paketi kuruyoruz : 

  `$ sudo apt-get install supervisor`

  Buradaki supervisor ayar dosyasını indirelim :  

  `$ wget http://aydintd.me/file/supervisor-ayar`

  Ve php.conf dosyasına ayar bilgilerini yükleyelim :  

  `$ install -m 644 supervisor-ayar /etc/supervisor/conf.d/php.conf`

  Supervisor un web sunucumuzda sürekli çalışması gerek.  
  Sürekli çalışır hale getirelim :  

  `$ sudo update-rc.d supervisor defaults`

        System start/stop links for /etc/init.d/supervisor already exist.  

  Supervisor prosesinin çalışıp çalışmadığına bakalım : 

  `$ sudo ps ax | grep supervisor`
      
        5469 ?        Ss     0:00 /usr/bin/python /usr/bin/supervisord
        5495 pts/0    S+     0:00 grep --color=auto supervisor  

  Buraya kadar bütün kurulumlarımız tamam. Tek yapmamız gereken gereksiz    
  nginx default ayarını sunucumuzdan kaldırmak :  

  `$ sudo rm -rf nginx-default/`

  Ve makinemizi yeniden başlatıyoruz.

  `$ sudo reboot`

  Bütün kurulum bu kadar. Bu işlemleri doğru ve düzgün sırayla  
  yapmışsanız; makinenizin içine wordpress, moodle vb. uygulamaları
  koyabilir, localinizde çalışma imkanına erişebilirsiniz.  

#### Lütfen Dikkat!

  Eğer bir web sunucu üzerinde çalışmıyorsanız sanal bir makine üzerinde  
  bu kurulumları gerçekleştirmeniz şiddetle tavsiye edilir.    

#### Dokümantasyonda kullanılan kaynaklar :  

  `http://gdemir.me`  
  `http://nginx.com/docs.html`  
