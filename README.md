
# ROCKY VE DEBIAN DAĞITIMLARINDA WORDPRESS KURULUMU

Bu Dökümantasyon debian ve rocky dağıtımlarında wordpress kurulumunu açıklar.
Debian ve Rocky için ayrı ayrı iki farklı dökümantasyon vardır.

Öncelikle bulunduğunuz ortamda internet bağlantısı olduğuna, sanal makineyi kurduğunuz program üstünden network portlarını düzgün verdiğinize ve IP almada sorun yaşamadığınızdan emin olun.

1. Kuruluma başlamadan önce firewall servisinden 80 ve 443 portuna izin verilmesi gerekiyor.

* ```systemctl status firewalld # servisin çalışıp çalışmadığını kontrol etmek için. ```
 Portları sırasıyla tcp protokolü ile açmak için
* ```firewall-cmd --add-port=80/tcp --permanent```
* ```firewall-cmd --add-port=443/tcp --permanent```
* ```firewall-cmd --reload # servisi reload etmek için```
* ```firewall-cmd --list-all # açılan portları görmek için```
* ```systemctl status firewalld # servis durumunu görmek için ```

2.  Wordpress Kurulumu dağıtımı için bir web servis(apache), database(mysql) ve backend kodlarının bulunduran bir yazılım dili (php) kuracağız.
* ```dnf install mysql # mysql kurmak için```
* ```dnf install mysql-server # mysql-server kurmak için```
* - ```systemctl status mysqld servisin durumunu görmek için```
* - ```systemctl enable --now mysqld # servisi çalıştırmak için```
* ```dnf install httpd # apache kurmak için ```
* - ```systemctl reload httpd # servisi yeniden başlatmak için```
* - ```systemctl status httpd # servis durumunu görmek için```
* Php kurabilmek için öncelikle güncel php_repolistini yüklemek zorundayız. [Güncel paketler her dağıtım için](https://rpms.remirepo.net/)
* - Günümüz için dnf install https://rpms.remirepo.net/enterprise/remi-release-9.rpm
* Kurduktan sonra aktif edilebilen modülleri listelemek için.
* - dnf module list php
 * - - Bende aktif edilen modül 8.3 olduğu için
 * - - ```dnf module enable php:remi-8.3```
* mysql ile php bağlantısı için
* - ``` dnf install php-mysqlnd ```
* Son php paketini de kuruyoruz. [işlevi](https://www.php.net/manual/en/install.fpm.php)
* - ```dnf install php-fpm ```
* - ```systemctl enable --now php-fpm # çalıştırmak için```

3. DATABASE Konfigrasyonu
* ```mysql_secure_installation #  MySQL veritabanı sunucusunu kurduktan sonra güvenlik önlemleri almak için kullanılan kullanılır.  ```

> [!IMPORTANT]
> Burada root ve kullanıcı için şifre girmeye özen gösterin çünkü bazı sistemlerde root şifresiz olunca mysql açılmıyor.

- - ```CREATE DATABASE db_name; # Databse oluşturmak için ```
* User oluşturmak ve şifre ataması yapmak için burada da şifre vermeyi unutmayın.
- -  ```CREATE USER 'db_user'@'localhost' IDENTIFIED BY 'db_password';```
* Kullanıcıya ayrıcalık tanıyıp databaseyi kullanmasını sağlıyoruz.
- - ```GRANT ALL PRIVILEGES ON db_name.* TO 'db_user'@localhost; ```
* Ayrıcalıkları güncelliyoruz.
- - ```FLUSH PRIVILEGES;```
- - ``` EXIT ```

4. Wordpressi wget kullanarak /var/www/ klasötrü içine kuruyoruz.
* ```cd /var/www/ ; wget https://wordpress.org/latest.tar.gz ```
* /var/www/ klasörü içine düzenli gözükmesi açısından sitemizin adında klasör ekliyoruz.
* - ```mkdir wp_first```;
* Arşiv dosyasını açıyoruz ve içindekileri wp_first klasörünün içine taşıyoruz.
* - ``` tar xf latest.tar.gz; mv ./wordpress/* ./wp_first/ ```
* Oluşturduğumuz klasöre ve içindeki dosyalara web servisi sahipliği atıyoruz (apache).
* - ```chown -R apache: ./wp_first/ ```
* /etc/httpd/conf.d/vhost_domain.com.conf komundaki dosyayının içine aşağıdaki texti yazıyoruz.
```
<VirtualHost *:80>
    ServerName www.wp_first
    DocumentRoot /var/www/wp_first
    <Directory /var/www/wp_first>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```
* apachede sorun olup olmadığını kontrol etmek için
* - ```apachectl -t```
* /etc/hosts dosyası içine girip ip adresimize sitemizi yönlendiriyoruz.
* Son olarak aşağıdaki komutu kullanıp dosyanın içine girdikten sonra database bilgilerimizi tabloya ekliyince wordpress ile oluşan site çalışmaya başlayacak.
* - ``` mv /var/www/html/wordpress/wp-config-sample.php /var/www/html/wordpress/wp-config.php ```
```
/** The name of the database for WordPress */
define( 'DB_NAME', 'db_name' );

/** Database username */
define( 'DB_USER', 'db_user' );

/** Database password */
define( 'DB_PASSWORD', 'db_password' );

/** Database hostname */
define( 'DB_HOST', 'localhost' );
```
* Servisleri yeniden başlatıyoruz ve sitemiz çalışmaya hazır.



