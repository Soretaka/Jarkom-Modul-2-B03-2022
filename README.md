# Jarkom-Modul-2-B03-2022

## Anggota Kelompok:

| Nama                           | NRP        |
| ------------------------------ | ---------- |
| Maheswara Danendra Satriananda | 5025201060 |
| James Silaban                  | 5025201169 |
| Anak Agung Yatestha Parwata    | 5025201234 |

## Pada praktikum kali ini, Maheswara mengerjakan soal 2,3,4 dan james mengerjakan soal 5,6,7 dan soal 1,8,9,10,11,12,13,14,15,16,dan 17 dikerjakan oleh yatestha.

### 1.WISE akan dijadikan sebagai DNS Master, Berlint akan dijadikan DNS Slave, dan Eden akan digunakan sebagai Web Server. Terdapat 2 Client yaitu SSS, dan Garden. Semua node terhubung pada router Ostania, sehingga dapat mengakses internet.

![image](https://user-images.githubusercontent.com/70903245/198835785-8c078f21-4b3f-4dc8-a495-b9463819c3c4.png)

Ostania sebagai router Network configuration:

```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
address 192.174.1.1
netmask 255.255.255.0

auto eth2
iface eth2 inet static
address 192.174.2.1
netmask 255.255.255.0

auto eth3
iface eth3 inet static
address 192.174.3.1
netmask 255.255.255.0
```

setup iptables:

```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.172.0.0/16
```

WISE sebagai DNS master Network configuration:

```
auto eth0
iface eth0 inet static
	address 192.174.2.2
	netmask 255.255.255.0
	gateway 192.174.2.1
```

Berlint sebagai DNS slave Network configuration:

```
auto eth0
iface eth0 inet static
	address 192.174.3.2
	netmask 255.255.255.0
	gateway 192.174.3.1
```

Eden sebagai web server Network configuration:

```
auto eth0
iface eth0 inet static
	address 192.174.3.3
	netmask 255.255.255.0
	gateway 192.174.3.1
```

SSS sebagai client Network configuration:

```
auto eth0
iface eth0 inet static
	address 192.174.1.2
	netmask 255.255.255.0
	gateway 192.174.1.1
```

Gardent sebagai client Network configuration:

```
auto eth0
iface eth0 inet static
	address 192.174.1.3
	netmask 255.255.255.0
	gateway 192.174.1.1
```

pada wise,berlint, dan eden akan diinstal:

```
apt update
apt install bind9 -y
```

pada sss dan gardent diinstal:

```
apt update
apt install dnsutils -y
```

### 2. Untuk mempermudah mendapatkan informasi mengenai misi dari Handler, bantulah Loid membuat website utama dengan akses wise.yyy.com dengan alias www.wise.yyy.com pada folder wise

Jalankan perintah berikut pada Eden:

```
echo '
zone "wise.f02.com" {
	type master;
	file "/etc/bind/wise/wise.f02.com";
};
' > /etc/bind/named.conf.local

mkdir /etc/bind/wise

echo '
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     wise.BO3.com. root.wise.B03.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      wise.B03.com.
@               IN      A       192.174.2.2
www             IN      CNAME   wise.B03.com.
@               IN      AAAA    ::1
' > /etc/bind/wise/wise.B03.com
```

hasil ping wise.B03.com

![image](https://user-images.githubusercontent.com/70903245/198836283-b8539688-6b89-466e-9122-a2d853eab1f6.png)

### 8. Pertama dengan webserver www.wise.yyy.com. Pertama, Loid membutuhkan webserver dengan DocumentRoot pada /var/www/wise.yyy.com

jalankan perintah berikut pada wise:

```
echo '
<VirtualHost *:80>
    <Directory /var/www/wise.B03.com>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
        # The ServerName directive sets the request scheme, hostname and port t$
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the requests Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/wise.B03.com
        ServerName wise.B03.com
        ServerAlias www.wise.B03.com

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
         CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
</VirtualHost>
# vim: syntax=apache ts=4 sw=4 sts=4 sr noet

' > /etc/apache2/sites-available/wise.B03.com.conf
a2ensite wise.B03.com
service apache2 reload
service apache2 restart

mkdir /var/www/wise.B03.com

wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=1S0XhL9ViYN7TyCj2W66BNEXQD2AAAw2e' -O /etc/apache2/sites-available/wise.zip
unzip /etc/apache2/sites-available/wise.zip
mv -T wise /var/www/wise.B03.com
```

hasil:

![image](https://user-images.githubusercontent.com/70903245/198836744-944bb85a-2318-4e48-b70b-1655c58cc07e.png)

### 9. Setelah itu, Loid juga membutuhkan agar url www.wise.yyy.com/index.php/home dapat menjadi menjadi www.wise.yyy.com/home

jalan command berikut di wise:

```
a2enmod rewrite
service apache2 restart
echo '
RewriteEngine on
RewriteRule ^home$ index.php [NC]
' > /var/www/wise.B03.com/.htaccess
```

hasil:

![image](https://user-images.githubusercontent.com/70903245/198836857-26bbdc09-3377-40d5-a3b0-ffe56fcf6ac1.png)

![image](https://user-images.githubusercontent.com/70903245/198836744-944bb85a-2318-4e48-b70b-1655c58cc07e.png)

### 10.11.12.13 Setelah itu, pada subdomain www.eden.wise.yyy.com, Loid membutuhkan penyimpanan aset yang memiliki DocumentRoot pada /var/www/eden.wise.yyy.com, Akan tetapi, pada folder /public, Loid ingin hanya dapat melakukan directory listing saja, Tidak hanya itu, Loid juga ingin menyiapkan error file 404.html pada folder /error untuk mengganti error kode pada apache. Loid juga meminta Franky untuk dibuatkan konfigurasi virtual host. Virtual host ini bertujuan untuk dapat mengakses file asset www.eden.wise.yyy.com/public/js menjadi www.eden.wise.yyy.com/js

jalankan perintah berikut di eden:

```
echo '
<VirtualHost *:80>
        <Directory /var/www/eden.wise.B03.com/public>
        	Options +Indexes
        </Directory>

        <Directory /var/www/eden.wise.B03.com/public/css/*>
                Options -Indexes
        </Directory>

        <Directory /var/www/eden.wise.B03.com/public/js/*>
                Options -Indexes
        </Directory>

	<Directory /var/www/eden.wise.B03.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>

        #untuk soal 13
        Alias "/js" "/var/www/eden.wise.B03.com/public/js"

        # The ServerName directive sets the request scheme, hostname and port t$
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the requests Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/eden.wise.B03.com
        ServerName eden.wise.B03.com
        ServerAlias www.eden.wise.B03.com


        RewriteEngine on
        RewriteCond %{HTTP_HOST} ^192\.174\.3\.3
        RewriteRule (.*) http://www.wise.B03.com/$1 [R=301,L]

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet

' > /etc/apache2/sites-available/eden.wise.B03.com.conf

a2ensite eden.wise.B03.com
service apache2 reload
service apache2 restart

mkdir /var/www/eden.wise.B03.com
wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=1q9g6nM85bW5T9f5yoyXtDqonUKKCHOTV' -O /etc/apache2/sites-available/eden.wise.zip
unzip "/etc/apache2/sites-available/eden.wise.zip"
mv -T eden.wise /var/www/eden.wise.B03.com

a2enmod rewrite
service apache2 restart

#untuk soal 12
echo '
ErrorDocument 404 /error/404.html

' > /var/www/eden.wise.B03.com/.htaccess
```

testing:

![image](https://user-images.githubusercontent.com/70903245/198837195-9b1182c7-4ba9-4395-8780-13b3ff0bc4de.png)

![image](https://user-images.githubusercontent.com/70903245/198837258-38065ee9-cf49-4ba5-878a-6ab04cfabcc1.png)

![image](https://user-images.githubusercontent.com/70903245/198837305-f1d1eedd-3419-48c7-b1ba-66a2e7a3ff66.png)

![image](https://user-images.githubusercontent.com/70903245/198837212-80543b34-7590-4acd-92f6-fee84ede0123.png)

## 14.15 Loid meminta agar www.strix.operation.wise.yyy.com hanya bisa diakses dengan port 15000 dan port 15500, dengan autentikasi username Twilight dan password opStrix dan file di /var/www/strix.operation.wise.yyy

jalankan perintah berikut di eden:

```

echo '
<VirtualHost *:15000> #soal 14
        <Directory /var/www/strix.operation.wise.B03.com>
                AuthType Basic
                AuthName "Restricted Content"
                AuthUserFile /var/www/strix.operation.wise.B03.com/.htpasswd
                Require valid-user
        </Directory>
        # The ServerName directive sets the request scheme, hostname and port t$
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the requests Host: header to
                # soal no 15
                AuthType Basic
                AuthName "Restricted Content"
                AuthUserFile /var/www/strix.operation.wise.B03.com/.htpasswd
                Require valid-user
        </Directory>
        # The ServerName directive sets the request scheme, hostname and port t$
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the requests Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/strix.operation.wise.B03.com
        ServerName strix.operation.wise.B03.com
        ServerAlias www.strix.operation.wise.B03.com
        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
                # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/strix.operation.wise.B03.com
        ServerName strix.operation.wise.B03.com
        ServerAlias www.strix.operation.wise.B03.com
        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
<VirtualHost *:15000> #soal no 14
        <Directory /var/www/strix.operation.wise.B03.com>
                AuthType Basic
                AuthName "Restricted Content"
                AuthUserFile /var/www/strix.operation.wise.B03.com/.htpasswd
                Require valid-user
        </Directory>
        # The ServerName directive sets the request scheme, hostname and port t$
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the requests Host: header to
                # soal no 15
                AuthType Basic
                AuthName "Restricted Content"
                AuthUserFile /var/www/strix.operation.wise.B03.com/.htpasswd
                Require valid-user
        </Directory>
        # The ServerName directive sets the request scheme, hostname and port t$
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the requests Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/strix.operation.wise.B03.com
        ServerName strix.operation.wise.B03.com
        ServerAlias www.strix.operation.wise.B03.com
        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
                # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/strix.operation.wise.B03.com
        ServerName strix.operation.wise.B03.com
        ServerAlias www.strix.operation.wise.B03.com
        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
' > /etc/apache2/sites-available/strix.operation.wise.B03.com.conf

a2ensite strix.operation.wise.B03.com
service apache2 reload
service apache2 restart
mkdir /var/www/strix.operation.wise.B03.com
wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=1bgd$
unzip "/etc/apache2/sites-available/wise.zip"
mv -T strix.operation.wise /var/www/strix.operation.wise.B03.com

#Untuk soal 14
echo '
Listen 80
Listen 15000
Listen 15500
' > /etc/apache2/ports.conf

a2enmod rewrite
service apache2 restart

#untuk soal 15
htpasswd -c -b /var/www/strix.operation.wise.B03.com/.htpasswd Twilight opStrix
service apache2 restart
```

hasil test:
tanpa port

![image](https://user-images.githubusercontent.com/70903245/198837668-66026840-7b4f-46a3-b223-ace428c2e929.png)

keredirect ke www.eden.B03.com

![image](https://user-images.githubusercontent.com/70903245/198837673-b11b420a-3490-486d-a9f8-d07e7850aba2.png)

dengan port 15000:

![image](https://user-images.githubusercontent.com/70903245/198837733-00282aca-c1ac-44ed-9ee6-7c9e821dee94.png)

diminta memasukan username password
![image](https://user-images.githubusercontent.com/70903245/198837737-7ded4935-8422-4b60-abc1-8a2f2d200dc0.png)

setelah memasukan twilight dan opStrix
![image](https://user-images.githubusercontent.com/70903245/198837750-8528fef5-517f-4c6b-975d-92908738e8b4.png)
