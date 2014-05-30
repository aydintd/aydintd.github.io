---
layout: post
title: "Wordpress on Dokku : Dokku Rehberi II"
date: 2014-05-30 01:54
comments: true
categories: [Teknik] 

---

Bu blog yazısı Dokku Rehberi'nin ikinci kısmı olup, bu yazıda Wordpress'in Dokku PaaS sistemlerine konuşlandırılmasını anlatacağım.

Yazarken Manowar - Warriors of the World dinledim, bazı kısımlar sert gelebilir. Like thunder from the sky...

Wordpress herkesin bildiği gibi PHP ile geliştirilmiş, bir çok veritabanı ile entegre çalışabilen bir CMS (Content Management System)
Bir önceki yazımda Dokku'nun nasıl kurulup, nasıl kullanılacağıyla ilgili anlatımı yapmıştım. (Şu anda kayıp, farkındayım) Bu yazıdaysa Wordpress'in taşı toprağıyla ilgileneceğim.

### Beklentiler ve Önbilgiler

Başlamadan önce;

*  Bir sanal makineye Dokku sistemi kurup, Network ve DNS ayarlarını lokaliniz için yapmanız bekleniyor. 

*  Sonrasında Wordpress'in Dokku için modifiye ettiğim deposunu kullanarak sistemi oluşturup, veritabanı olarak Mysql kullanacağız.

## Dokku Eklentileri
 
PaaS sistemlerde veritabanları eklentiler olarak web uygulamasına bağlanır. Aynı Heroku'nun Postgresql eklentisini uygulumanıza sonradan eklemeniz gibi. Bu durumu Dokku için özelleştirecek olursak, izole konteynırlarda tutulan uygulamanıza, başka bir izole konteynır içerisine kurulan veritabanınız bağlanır ve Dokku tarafından haberleştirilmesi sağlanır. Böylece uygulama ve veritabanını birbirinden ayırmış iki izole linux konteynırında tutulmuş olur.

Bu yazıda Wordpress üzerinden gidecek olsam da, diğer web uygulamaları için izlenecek yol Dokku özelinde benzer. Bu yüzden kurmuş olduğunuz Dokku sisteme kuracağınız eklentilerle daha fazla güç kazandırmak hedefindeyiz. PhpMyAdmin'cilere selam olsun!

### Mysql Eklentisi 

Dokku'ya Mysql kurmak için şu abimizin(k2nr) deposundaki eklentiyi aşağıdaki şekilde sunucuya kuruyoruz :

	$ cd /var/lib/dokku/plugins
   	$ git clone https://github.com/k2nr/dokku-mysql-plugin mysql
   	$ chmod +x mysql/install mysql/commands
   	$ dokku plugins-install

Kurulum bittikten sonra `$ dokku help` komutuyla Dokku'ya Mysql ile ilgili özel güçler verdiğinizi görebilirsiniz :

   	...
   	mysql:create <app>     Create a MySql container
   	mysql:delete <app>     Delete specified MySql container
   	mysql:info <app>       Display database informations
   	mysql:link <app> <db>  Link an app to a MySql database
   	mysql:logs <app>       Display last logs from MySql container
   	...

Mysql eklentisi kurulduktan sonra bu eklenti bize WordPress için gereken veritabanı konteynırını oluşturmamıza yarayacak.

### Dokku User-Env Compile Eklentisi

Bu eklenti ise Mysql konteynırıyla WordPress arasındaki bağlantıyı kurmak için gereken ortam değişkenlerini konteynırlar arası kullanabilmek için gerekiyor. 

Kurulumu şu şekilde :

	$ cd /var/lib/dokku/plugins
	$ git clone https://github.com/musicglue/dokku-user-env-compile.git user-env-compile
	$ dokku plugins-install

Mysql konteynırı oluşturduğumuzda, eklenti bize WordPress'te tanımlamamız gereken veritabanı kullanıcısı, veritabanının adı, parolası gibi bilgileri ortam değişkenleri vasıtasıyla verecek. Biz de bu eklenti sayesinde bilgileri WordPress'e geçirip, ekstra işlem yapmadan WordPress'i kuracağız.

## WordPress'i Dokku'ya Konuşlandırma

Daha önceden modifiye ettiğim Wordpress deposunu yerelinize çekin :

	$ git clone git@github.com:aydintd/WordPress.git wp  # Uygulamanın ismi wp olacak buraya dikkat.

Modifiye'nin sebebi `wp-config.php` dosyasında yaptığım ortam değişkenlerini ortamdan almak için yaptığım düzenleme. (Bkz. PHP'de getenv() fonksiyonu)

Şimdi uzak yolu WordPress için Dokku'ya tanımlayıp, uygulamayı push etmek :

	$ cd wp
	$ git remote add dokku dokku@dokku.me:wp  # dokku.me sanalda önceden kurduğum Dokku makinesinin adresi, kafanız karışmasın.
	$ git push dokku master

Dokku, Heroku Buildpackleri sayesinde WordPress'in bir PHP uygulaması olduğunu anlayıp, Dokku'ya konuşlandıracak. Ben bu örnek için uygulama ismine `wp` adını vermiştim. Konuşlandırma işlemi bittiğinde ve şafakta `http://wp.dokku.me` adresine bakın, `Database connection error` görüyorsanız anlayın ki Gandalf'ın gelmesi yakın. 

Şimdiyse veritabanını wp uygulaması için oluşturalım :

	$ ssh dokku@dokku.me dokku mysql:create wp
	
	-----> MySql container created: mysql/wp
	    01d4a5471d35

            Host: 172.17.0.3		          
	    User: 'admin'			        
	    Password: 'hFtI5DtKEa1n4KxM'
	    Database: 'db'
	    Internal port: 3306

Gördüğünüz üzere Mysql konteynırı da oluştu ve bize gerekli bilgiler verildi. Ancak bu bilgileri elle girmek yerine Dokku bunu kurduğumuz user-env eklentisiyle kendisi halledebiliyor. Wordpress'i de bu yüzden modifiye ettim. Bi' şey yapmıyorsunuz yani.

Şimdiyse tarayıcınıza `http://wp.dokku.me` yazdığınızda sizi klasik Wordpress kurulum ekranı karşılayacak. PhpMyAdmin'cilere yine selam olsun!

## Dokku Debugging Style

Dokku çok güçlü bir PaaS altyapısı bize sunuyor. Bu örnek özelinde yukarıdaki anlatımları uyguladığınızda başarıya ulaşamamışsanız benim karşılaştığım ve çözdüğüm bir kaç yöntemden size bahsedeyim.

*  Uygulamalarınız konuşlanamamış olabilir. Dokku sunucuda :

        $ sudo docker ps 

        CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                     NAMES
        01d4a5471d35        mysql/wp:latest     /usr/sbin/mysqld       7 minutes ago       Up 7 minutes        0.0.0.0:49197->3306/tcp   berserk_brattain       
        7ac0cb59bf18        dokku/wp:latest     /bin/bash -c '/start   About an hour ago   Up About an hour    0.0.0.0:49196->5000/tcp   romantic_pasteur     

Bu şekilde bir çıktı gözlemlemeniz beklenmekte. Bu çıktıyı göremiyorsanız yukarıdaki adımları takip edin.

*   Uygulamanızla veritabanı sunucunuz haberleşemiyor olabilir. Bu durumda tarayıcıda `Database Connection Error` gözlemlemeniz beklenmekte.

Bu durumun çeşitli sebepleri olabileceği gibi en çok karşılaştığım durum Mysql tarafından oluşturulan ortam değişkenlerinin WordPress'e düzgün geçirilememesi. Şu adımları izlerseniz fikir sahibi olabilirsiniz :

Dokku sunucuda :

	$ sudo dokku config wp  # wp uygulamasına geçirilen ortam değişkenlerini listeler, örnek çıktı :

	=== wp config vars ===
	DB_HOST: 172.17.0.3
	DB_NAME: db
	DB_PASS: Wdui95G7Jgv7tBkY
	DB_PORT: 3306
	DB_USER: admin

Bu değişkenlerle Wordpress'inizin `wp-config.php` dosyasındaki değişkenlerin birebir aynı olduğunu kontrol edin :

	$ sudo dokku run wp 'cat wp-config.php'   # evet konuşlandırılan konteynırın içerisinde de komut çalıştırabiliyorsunuz!

	// ** MySQL settings - You can get this info from your web host ** //
	/** The name of the database for WordPress */
	define('DB_NAME', getenv('DB_NAME'));

	/** MySQL database username */
	define('DB_USER', getenv('DB_USER'));

	/** MySQL database password */
	define('DB_PASSWORD', getenv('DB_PASS'));

	/** MySQL hostname */
	define('DB_HOST', getenv('DB_HOST').':'.getenv('DB_PORT'));

	/** Database Charset to use in creating database tables. */
	define('DB_CHARSET', 'utf8');
	...

Gerisi sizin hayal gücünüze kalıyor. Hepsi bu kadar. :-)
	
