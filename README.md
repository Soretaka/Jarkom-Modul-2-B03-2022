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

### 3. Setelah itu ia juga ingin membuat subdomain eden.wise.yyy.com dengan alias www.eden.wise.yyy.com yang diatur DNS-nya di WISE dan mengarah ke Eden

Untuk membuat subdomain, kita harus mengedit file `/etc/bind/wise/wise.yyy.com` dengan menambahkan subdomai **eden** dan mengarah pada **192.174.3.3** (IP Eden) buat aliasnya dengan CNAME. Kita dapat melakukan hal tersebut dengan command
```
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
eden            IN      A       192.174.3.3
www             IN      CNAME   wise.B03.com.
www.eden        IN      CNAME   eden.wise.B03.com.
ns1             IN      A       192.174.3.3
operation       IN      NS      ns1 
' > /etc/bind/wise/wise.B03.com
```

Kemudian restart service bind

```
service bind9 restart
```

Lalu coba ping ke subdomain pada client SSS

![image](https://user-images.githubusercontent.com/78299006/199212433-c3c1ae47-1d2a-4d5d-987f-620ea2e560f2.png)

### 4.  Buat juga reverse domain untuk domain utama

Edit file `/etc/bind/named.conf.local` di *WISE* dengan menambahkan reverse 3 byte awal dari IP *Wise*. Edit file tersebut dapat dilakukan dengan command

```
echo '
zone "wise.B03.com" {
        type master;
        notify yes;
        also-notify { 192.174.3.2; };
        allow-transfer { 192.174.3.2; };
        file "/etc/bind/wise/wise.B03.com";
};

zone "2.174.192.in-addr.arpa" {
        type master;
        file "/etc/bind/wise/2.174.192.in-addr.arpa";
};
' > /etc/bind/named.conf.local
```

Kemudian, edit file `/etc/bind/wise/[Reverse dari 3 Byte awal].in-addr.arpa` dengan command berikut:

```
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
2.174.192.in-addr.arpa. IN      NS      wise.B03.com.
2                       IN      PTR     wise.B03.com.
' > /etc/bind/wise/2.174.192.in-addr.arpa
```

Restart bind9

```
service bind9 restart
```

Kemudian cek pada client *SSS* apakah konfigurasi sudah benar. Pastikan telah update dan install dnsutil pada client *SSS*

![image](https://user-images.githubusercontent.com/78299006/199213974-ba4a5c05-d2ea-4529-948f-323ca6037ae3.png)




### 5.  Agar dapat tetap dihubungi jika server WISE bermasalah, buatlah juga Berlint sebagai DNS Slave untuk domain utama

Pada *Wise*, edit `zone "wise.yyy.com"` pada `/etc/bind/named.conf.local` dengan command berikut:

```
echo '
zone "wise.B03.com" {
        type master;
        notify yes;
        also-notify { 192.174.3.2; };
        allow-transfer { 192.174.3.2; };
        file "/etc/bind/wise/wise.B03.com";
};'
> /etc/bind/named.conf.local
```
Kemudian restart bind9

```
service bind9 restart
```

Kemudian buka *Berlint*, lalu update package lists dan instal aplikasi bind9 pada *Berlint*

```
apt-get update
apt-get install bind9 -y
```

Edit file `/etc/bind/named.conf.local` pada *Berlint* dengan command berikut:

```
echo '
zone "wise.B03.com" {
        type slave;
        masters { 192.174.2.2; };
        file "var/lib/bind/wise.B03.com";
};

' > /etc/bind/named.conf.local
```

Lakukan restart bind9 pada *Berlint*

```
service bind9 restart
```

Untuk testing, matikan service bind9 pada *Wise*

```
service bind9 stop
```

Pada client *SSS*, masukkan nameserver mengarah ke IP *Wise* dan *Berlint* dengan command:

```
echo '
nameserver 192.174.2.2 #IP server Wise
nameserver 192.174.3.2 #IP server Berlint
' > /etc/resolv.conf
```

kemudian periksa dengan ping www.wise.yyy.com pada client *SSS*

![image](https://user-images.githubusercontent.com/78299006/198867497-8b0e9e06-dd7b-4b93-b85d-72fae36390fc.png)

Jika berhasil melakukan ping ke www.wise.yyy.com, maka DNS Slave berhasil dibuat


### 6.  Karena banyak informasi dari Handler, buatlah subdomain yang khusus untuk operation yaitu operation.wise.yyy.com dengan alias www.operation.wise.yyy.com yang didelegasikan dari WISE ke Berlint dengan IP menuju ke Eden dalam folder operation

Edit file `/etc/bind/wise/wise.yyy.com` dengan command berikut. 
```
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
eden            IN      A       192.174.3.3
www             IN      CNAME   wise.B03.com.
www.eden        IN      CNAME   eden.wise.B03.com.
ns1             IN      A       192.174.3.3 ; IP Eden
operation       IN      NS      ns1 
' > /etc/bind/wise/wise.B03.com
```

kemudian pada file `/etc/bind/named.conf.options`, comment `dnssec-validation auto;` tambahkan `allow-query{any;};`, seperti command berikut:

```
echo '
options {
    directory "/var/cache/bind";

    //forwarders {
    //    0.0.0.0;
    //};

    //dnssec-validation auto;
    allow-query { any; };
    auth-nxdomain no;    # conform to RFC1035
    listen-on-v6 { any; };
};
' > /etc/bind/named.conf.options
```

Tambahkan juga `allow-transfer { "IP Berlint"; };` pada file `/etc/bind/named.conf.local ` (telah dilakukan di nomor 5)

Setelah itu restart bind9 
```
service bind9 restart
```

Pada server *Berlint*, edit file `/etc/bind/named.conf.options`, comment `dnssec-validation auto;` dan tambahkan `allow-query{any;};`, seperti command berikut:
```
echo '
options {
        directory "/var/cache/bind";
        // forwarders {
        //  0.0.0.0;
        // }; 

        // dnssec-validation auto;
        allow-query{any;};

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};' > /etc/bind/named.conf.options
```
Juga edit file `/etc/bind/named.conf.local` di *Berlint* dengan menambahkan `zone "operation.wise.yyy.com"`

```
echo '
zone "operation.wise.B03.com" {
        type master;
        file "/etc/bind/operation/operation.wise.B03.com";
};

zone "wise.B03.com" {
        type slave;
        masters { 192.174.2.2; };
        file "var/lib/bind/wise.B03.com";
};

' > /etc/bind/named.conf.local
```

Buat direktori baru bernama **operation** dan edit file **operation.wise.yyy.com**

```
echo '
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     operation.wise.BO3.com. root.operation.wise.B03.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      operation.wise.B03.com.
@               IN      A       192.174.3.3 ; IP Eden
www             IN      A       192.174.3.3 ; IP Eden
' > /etc/bind/operation/operation.wise.B03.com
```

Restart bind9

```
service bind9 restart
```

Untuk pengujian, lakukan ping ke domain **www.operation.wise.yyy.com** dari client *SSS*

![image](https://user-images.githubusercontent.com/78299006/198870425-3fbf6d24-bc44-4588-9fca-6e7297fbb021.png)


### 7.  Untuk informasi yang lebih spesifik mengenai Operation Strix, buatlah subdomain melalui Berlint dengan akses strix.operation.wise.yyy.com dengan alias www.strix.operation.wise.yyy.com yang mengarah ke Eden 

Untuk membuat subdomain pada *Berlint*, edit file **operation.wise.yyy.com** dan tambahkan strix sebagai subdomain seperti command berikut:

```
echo '
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     operation.wise.BO3.com. root.operation.wise.B03.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      operation.wise.B03.com.
@               IN      A       192.174.3.3
www             IN      A       192.174.3.3
strix           IN      A       192.174.3.3 ; IP Eden
www.strix       IN      CNAME   strix.operation.wise.B03.com.
' > /etc/bind/operation/operation.wise.B03.com
```

setelah itu, restart bind

```
service bind9 restart
```

kemudian lakukan ping ke subdomain 

![image](https://user-images.githubusercontent.com/78299006/198870659-6205446d-1477-4f0e-8405-bfed9d5f4624.png)


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

### 16. dan setiap kali mengakses IP Eden akan dialihkan secara otomatis ke www.wise.yyy.com

tambahkan

```
 RewriteEngine on
        RewriteCond %{HTTP_HOST} ^192\.174\.3\.3
        RewriteRule (.*) http://www.wise.B03.com/$1 [R=301,L]
```

pada /etc/apache2/sites-available/eden.wise.B03.com.conf pada node Eden didalam configurasi virtual hostnya

hasil:

![image](https://user-images.githubusercontent.com/70903245/198837951-dd91bb09-7665-47ac-b31b-185f3f2d6653.png)

![image](https://user-images.githubusercontent.com/70903245/198837986-e6391cc1-d3fe-4869-9503-7145e4d5047e.png)

### 17. Karena website www.eden.wise.yyy.com semakin banyak pengunjung dan banyak modifikasi sehingga banyak gambar-gambar yang random, maka Loid ingin mengubah request gambar yang memiliki substring "eden" akan diarahkan menuju eden.png

Jalankan script dibawah ini pada eden:

```
echo '
RewriteEngine On
RewriteRule ^(.*)eden(.*)\.jpg$ http://www.eden.wise.B03.com/public/images/eden.png [L,R=301]
ErrorDocument 404 /error/404.html
' > /var/www/eden.wise.B03.com/.htaccess
```

```
 ^(.*)eden(.*)\.jpg$
```

merupakan regex untuk mendapatkan substring eden dan gambar

hasil:
![image](https://user-images.githubusercontent.com/70903245/198838203-f8711e01-d833-40eb-90f4-6ef3def22a1e.png)

![image](https://user-images.githubusercontent.com/70903245/198838243-3a297ecd-3ddf-4046-b52c-05f4208d2fde.png)

## Kendala yang dialami

ketika melakukan apt-get install apapun, koneksinya lambat, sekali apt-get install bisa berlangsung kurang lebih 15 menit
