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
