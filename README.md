# Jarkom-Modul-2-B03-2022

## Anggota Kelompok:

| Nama                           | NRP        |
| ------------------------------ | ---------- |
| Maheswara Danendra Satriananda | 5025201060 |
| James Silaban                  | 5025201169 |
| Anak Agung Yatestha Parwata    | 5025201234 |

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

Jalankan perintah berikut:

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
