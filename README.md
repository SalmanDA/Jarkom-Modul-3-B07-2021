# Jarkom-Modul-3-B07-2021

**Anggota kelompok**:

- Salman Damai Alfariq (05111840000159)
- Ridho Ajiraga Jagiswara (05111940000170)
- David Ralphwaldo Martuaraja (05111940000190)

---

## Prefix IP

Prefix IP dari kelompok kami adalah `192.180`

## Soal 1

Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server 

- Pertama, memberi **Foosha** konfigurasi sehingga dapat memberikan internet kepada **EniesLobby**, **Jipangu**, dan **Water7**
    - Network Configuration (**Foosha**)
        
        ```
        auto eth0
        iface eth0 inet dhcp
        
        auto eth1
        iface eth1 inet static
        	address 192.180.1.1
        	netmask 255.255.255.0
        
        auto eth2
        iface eth2 inet static
        	address 192.180.2.1
        	netmask 255.255.255.0
        
        auto eth3
        iface eth3 inet static
        	address 192.180.3.1
        	netmask 255.255.255.0
        ```
        
    - Agar komputer di bawahnya bisa mendapatkan internet
        
        ```
        iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.180.0.0/16
        ```
        
    - Cek nameserver: hasil → 192.168.122.1 (nameserver foosha)
        
        ```
        cat /etc/resolv.conf
        ```
        
- Konfigurasi **EniesLobby**, **Jipangu**, dan **Water7**
    - EniesLobby → DNS Server
        
        ```
        # network configuration
        iface eth0 inet static
          address 192.180.2.2
          netmask 255.255.255.0
          gateway 192.180.2.1
        ```
        
        save di `/root/.bashrc`
        
        ```
        echo nameserver 192.168.122.1 > /etc/resolv.conf
        apt-get update
        apt-get install nano
        apt-get install bind9
        ```
        
    - Jipangu → DHCP Server
        
        ```
        # network configuration
        auto eth0
        iface eth0 inet static
          address 192.180.2.4
          netmask 255.255.255.0
          gateway 192.180.2.1
        ```
        
        save  di `/root/.bashrc`
        
        ```
        echo nameserver 192.168.122.1 > /etc/resolv.conf
        apt-get update
        apt-get install nano
        apt-get install isc-dhcp-server
        dhcpd --version
        ```
        
        menentukan interface yang akan dilayani DHCP server
        
        ```
        nano /etc/default/isc-dhcp-server
        ```
        
        ubah menjadi
        
        ```
        INTERFACES="eth0"
        ```
        
    - Water7 → Proxy Server
        
        ```
        # network configuration
        auto eth0
        iface eth0 inet static
          address 192.180.2.3
          netmask 255.255.255.0
          gateway 192.180.2.1
        ```
        
        save di `/root/.bashrc`
        
        ```
        echo nameserver 192.168.122.1 > /etc/resolv.conf
        apt-get update
        apt-get install nano
        apt-get install squid
        ```
        
        ![Untitled](Modul%203%20f647bcde7df740e18d77f6606b0995c7/Untitled.png)
        

## Soal 2

Foosha sebagai DHCP Relay

- Foosha → install DHCP relay
    
    ```
    iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.180.0.0/16
    apt-get update
    apt-get install nano
    apt-get install isc-dhcp-relay -y
    ```
    
- Isikan konfigutrasi seperti ini pada `/etc/default/isc-dhcp-relay`
    
    ![Untitled](Modul%203%20f647bcde7df740e18d77f6606b0995c7/Untitled%201.png)
    
    - start dhcp relay → `service isc-dhcp-relay start`

## Soal 3

Ada beberapa kriteria yang ingin dibuat oleh Luffy dan Zoro, yaitu:

- Semua client yang ada **HARUS** menggunakan konfigurasi IP dari DHCP Server.
    
    ```
    # network configuration -> loguetown, alabasta, tottoland, skypie
    auto eth0
    iface eth0 inet dhcp
    ```
    
- Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.20 - [prefix IP].1.99 dan [prefix IP].1.150 - [prefix IP].1.169 → `nano /etc/dhcp/dhcpd.conf` (**Jipangu**)
    
    ```
    subnet 192.180.2.0 netmask 255.255.255.0 {
        option routers 192.180.2.1;
    }
    ```
    
    ```
    subnet 192.180.1.0 netmask 255.255.255.0 {
        range 192.180.1.20 192.180.1.99;
        range 192.180.1.150 192.180.1.169;
        option routers 192.180.1.1;
        option broadcast-address 192.180.1.255;
        option domain-name-servers 192.180.2.2;
        default-lease-time 360;
        max-lease-time 7200;
    }
    ```
    
    - restart dhcp server → `service isc-dhcp-server restart` → `service isc-dhcp-server status`
    - Untuk mengecek, jalankan `ip a` pada Loguetown.
    
    ![Untitled](Modul%203%20f647bcde7df740e18d77f6606b0995c7/Untitled%202.png)

## Soal 4

Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.30 - [prefix IP].3.50

- tambah ini di → `nano /etc/dhcp/dhcpd.conf` (Jipangu)
    
    ```
    subnet 192.180.3.0 netmask 255.255.255.0 {
        range 192.180.3.30 192.180.3.50;
        option routers 192.180.3.1;
        option broadcast-address 192.180.3.255;
        option domain-name-servers 192.180.2.2;
        default-lease-time 720;
        max-lease-time 7200;
    }
    ```
    
    - restart dhcp server → `service isc-dhcp-server restart` → `service isc-dhcp-server status`
    - Untuk mengecek, jalankan `ip a` pada TottoLand
    
    ![Untitled](Modul%203%20f647bcde7df740e18d77f6606b0995c7/Untitled%203.png)
    

## Soal 5

Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut.

- Tambahkan DNS Forwarder di EniesLobby
    
    `nano /etc/bind/named.conf.options` (EniesLobby)
    
    ```
    # uncomment dan tambah
    forwarders {
        192.168.122.1;
    };
    
    # comment
    // dnssec-validation auto;
    
    # tambahkan
    allow-query{any;};
    ```
    
    ![Untitled](Modul%203%20f647bcde7df740e18d77f6606b0995c7/Untitled%204.png)
    
    - restart bind9 (EniesLobby) → `service bind9 restart`

## Soal 6

Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit. *(sudah sekaligus dikerjakan pada nomor 3 dan 4)*

- Switch 1: 6 menit → 360 detik
- Switch 3: 12 menit → 720 detik
- Max: 120 menit → 7200 detik
- SS dari `cat /etc/dhcp/dhcpd.conf` (Jipangu)
    
    ![Untitled](Modul%203%20f647bcde7df740e18d77f6606b0995c7/Untitled%205.png)
    

## Soal 7

Luffy dan Zoro berencana menjadikan **Skypie** sebagai server untuk jual beli kapal yang dimilikinya dengan **alamat IP yang tetap** dengan IP [prefix IP].3.69

- cek hwadress di **Skypie → `ip a`**
    
    ![Untitled](Modul%203%20f647bcde7df740e18d77f6606b0995c7/Untitled%206.png)
    
    - hwadress **Skypie** → `ae:ac:30:1c:73:be`
- tambah config dhcp **Jipangu →** `nano /etc/dhcp/dhcpd.conf`
    
    ```
    # tambah
    host Skypie {
        hardware ethernet ae:ac:30:1c:73:be;
        fixed-address 192.180.3.69;
    }
    ```
    
    - restart dhcp server → `service isc-dhcp-server restart` → `service isc-dhcp-server status`
- tambah network config di **Skypie →** `nano /etc/network/interfaces`
    
    ```
    # tambah
    hwaddress ether ae:ac:30:1c:73:be
    ```
    
    ![Untitled](Modul%203%20f647bcde7df740e18d77f6606b0995c7/Untitled%207.png)
    
- restart **Skypie,** hasilnya (`ip a`):
    
    ![Untitled](Modul%203%20f647bcde7df740e18d77f6606b0995c7/Untitled%208.png)
    
## Soal 8

**Loguetown** digunakan sebagai client **Proxy** agar transaksi jual beli dapat terjamin keamanannya, juga untuk mencegah kebocoran data transaksi.
Pada Loguetown, proxy **harus bisa diakses** dengan nama [**jualbelikapal.yyy.com](http://jualbelikapal.yyy.com/)** dengan port yang digunakan adalah **5000**

- Buat domain mengarah ke proxy di DNS
    
    `/etc/bind/named.conf.local` di **EniesLobby**
    
    ```
    zone "jualbelikapal.b07.com" {
    	type master;
    	file "/etc/bind/jarkom/jualbelikapal.b07.com";
    };
    ```
    
- (**EniesLobby**) Buat direktori bernama jarkom, copy `db.local` dan beri nama `jualbelikapal.b07.com`
    
    ```
    mkdir /etc/bind/jarkom
    cp /etc/bind/db.local /etc/bind/jarkom/jualbelikapal.b07.com
    ```
    
- (**EniesLobby**) Tambahkan konfigurasi pada `/etc/bind/jarkom/jualbelikapal.b07.com`, seperti gambar di bawah ini
    
    ![Untitled](Modul%203%20f647bcde7df740e18d77f6606b0995c7/Untitled%209.png)
    
- Setelah domain berhasil dibuat, buat proxy. Pada **Water7**, lakukan konfigurasi pada file `/etc/squid/squid.conf`
    
    ```
    http_port 5000
    visible_hostname Water7
    
    http_access allow all
    ```
    
- Restart squid → `service squid restart`
- Pada **Loguetown**, aktifkan proxy → `export http_proxy="http://jualbelikapal.b07.com:5000"`
    
    kemudian cek dengan `lynx http://its.ac.id` .
    
    ![Untitled](Modul%203%20f647bcde7df740e18d77f6606b0995c7/Untitled%2010.png)
    

## Soal 9

Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy ****dipasang **autentikasi user proxy dengan enkripsi MD5** dengan **dua username,** yaitu luffybelikapalyyy dengan password ****luffy_yyy **dan** zorobelikapalyyy dengan password zoro_yyy

- Pada **Water7**, jalankan:
    
    `apt-get update`
    
    `apt-get install apache2-utils -y`
    
    `touch /etc/squid/passwd`
    
    `htpasswd -m /etc/squid/passwd luffybelikapalb07`
    
    - kemudian masukkan password → `luffy_b07`
    
    `htpasswd -m /etc/squid/passwd zorobelikapalb07`
    
    - kemudian masukkan password → `zoro_b07`
- Kemudian, edit file `/etc/squid/squid.conf`.
    
    ```
    http_port 5000
    visible_hostname Water7
    
    auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
    auth_param basic children 5
    auth_param basic realm Proxy
    auth_param basic credentialsttl 2 hours
    auth_param basic casesensitive on
    acl USERS proxy_auth REQUIRED
    http_access allow USERS
    ```
    
- Restart squid → `service squid start`.
- `lynx [http://its.ac.id](http://its.ac.id)` sebagai **Loguetown**
    
    ![Untitled](Modul%203%20f647bcde7df740e18d77f6606b0995c7/Untitled%2011.png)
    

## Soal 10

Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari **Senin-Kamis pukul 07.00-11.00** dan setiap hari **Selasa-Jum’at pukul 17.00-03.00** keesokan harinya **(sampai Sabtu pukul 03.00)**

- Pada **Water7**, ubah `/etc/squid/acl.conf` menjadi:
    
    ```
    acl AVAILABLE_WORKING time MTWH 07:00-11:00
    acl AVAILABLE_WORKING1 time TWHF 17:00-24:00
    acl AVAILABLE_WORKING2 time WHFA 00:00-03:00
    ```
    
- edit `/etc/squid/squid.conf` menjadi:
    
    ```
    include /etc/squid/acl.conf
    
    http_port 5000
    visible_hostname Water7
    
    auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
    auth_param basic children 5
    auth_param basic realm Proxy
    auth_param basic credentialsttl 2 hours
    auth_param basic casesensitive on
    acl USERS proxy_auth REQUIRED
    http_access allow AVAILABLE_WORKING USERS
    http_access allow AVAILABLE_WORKING1 USERS
    http_access allow AVAILABLE_WORKING2 USERS
    
    http_access deny all
    ```
    
- restart squid → `service squid restart`
- untuk mengecek, gunakan sintaks dibawah ini untuk mengubah date time
    - `date -s "12 NOV 2021 16:00:00"` (access denied)
    - `date -s "12 NOV 2021 18:00:00"` (access accepted)
- coba akses `lynx http://its.ac.id` melalui **Loguetown** diluar waktu AVAILABLE_WORKING (jalankan `date -s "12 NOV 2021 16:00:00"`)**,** maka akan muncul `Access Denied` seperti gambar dibawah ini
    
    ![Untitled](Modul%203%20f647bcde7df740e18d77f6606b0995c7/Untitled%2012.png)
