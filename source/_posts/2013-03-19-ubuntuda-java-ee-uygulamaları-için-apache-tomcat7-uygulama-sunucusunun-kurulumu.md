---
title: "Ubuntuda Java EE uygulamaları için Apache tomcat7 Uygulama Sunucusunun Kurulumu"
layout: post
date: 2013-02-26 23:34
comments: true
categories: [Teknik]
---
  
  Java2EE web uygulamaları için gereken açık kaynak apache-tomcat7 uygulama  
  sunucusunun kurulumunu adım adım takip edelim :

### Başlarken  

  Kurulum Linux Ubuntu 12.10 32bit işletim sistemi üzerinde Apache Tomcat 7.0.30  
  sürümü olarak sizi yönlendirecektir.    

  *  Not : Tomcat Uygulama sunucusunu kurmadan, makinenizde Java nın kurulu  
  olduğundan emin olun.  

### Kurulum  

  Aşağıdaki komutları çalıştırın :  

  `$ sudo apt-get update && sudo apt-get install tomcat7`

  Böylece tomcat7 uygulama sunucusunu makinemize kurmuş olacağız.  

<!--more-->

### Konfigurasyon  

  tomcat7 sunucusunu konfigurasyona başlamak için durduralım :

  `$ sudo service tomcat7 stop`

  `> Stopping Tomcat servlet engine tomcat7 [ OK ]`

  Favori text editörünüzle tomcat7 in ayar dosyasını açıp içerisindeki commentli halde bulunan (#)  
  JAVA_HOME dosya yolununun commentini kaldırıp aşağıdaki hale getirin.  

  `$ sudo vim /etc/default/tomcat7`

  `JAVA_HOME=/usr/lib/jvm/jdk1.7.0`

  *  Not : Makinenizde kurulu olan JDK nın dosya yolunu doğru verdiğinizden emin  
  olun. Aksi halde uygulama sunucunuz başlayamayacaktır.  

  Sunucumuzu tekrar başlatalım :  

  `$ sudo service tomcat7 start`

  Tomcat için Firewall unuzda ön tanımlı bir port açın :

  `$ sudo ufw enable 8080/tcp`
 
  Sunucunun JDK üzerinde çalıştığından emin olalım :

  `$ /usr/share/tomcat7/bin/version.sh`

  Aşağıdaki gibi JVM çıktılarını gözlemlemelisiniz.  

        Using CATALINA_BASE:   /usr/share/tomcat7  	  
        Using CATALINA_HOME:   /usr/share/tomcat7  	  
        Using CATALINA_TMPDIR: /usr/share/tomcat7/temp  	  
        Using JRE_HOME:        /usr  	  
        Using CLASSPATH:  	
        /usr/share/tomcat7/bin/bootstrap.jar:/usr/share/tomcat7/bin/tomcat-juli.jar  	 
        Server version: Apache Tomcat/7.0.30  		
        Server built:   Jan 10 2013 04:10:25  		
        Server number:  7.0.30.0  	
        OS Name:        Linux  		
        OS Version:     3.5.0-25-generic    
        Architecture:   i386  
        JVM Version:    1.7.0-b147  
        JVM Vendor:     Oracle Corporation  
  
### Son  

  Web Browser a `http://localhost:8080` yazdığınızda `It Works!` gibi bir yazı    
  görüyorsanız, tomcat7 uygulama sunucu kurulumunu başarıyla gerçekleştirmişsinizdir.  

  * Önemli not : Kurulumdan hemen sonra Eclipse editörü üzerinde bir  
  Java EE uygulamasını görüntülemek istiyorsanız, tomcat7 sunucunuzu konsoldan tekrar kapatmanız gerekebilir.  
  Uygulamayı görüntülemeye çalıştığınızda tomcat7 in başka bir prosesle meşgul olduğu  
  mesajıyla karşı karşıya kalırsanız,tomcat7 sunucusunu elle kapatın ve Eclipse editörü yardımıyla 
  uygulamanızı tekrar sunucu üzerinde görüntülemeye çalışın.(çalıştırın)      

### Kaynakça   

  `http://hendrelouw73.wordpress.com`  
  `blog.eviac.com`
