Sisi Server CHR nya : 
1. buat SSTP = (PPP - SSTP SERVER - enable )

2. buat scret nya = (PPP - screts - name : kotaA - password : bebas - local adress : 10.10.10.1 -
remote adress : 10.10.10.2 ) -> buatkan juga untuk kotaB ( Remote Adress : 10.10.10.3- sisanya sama)

3. buat ip adress = (IP - adresess - klik + - adresss : 10.10.40.1/24 - interface: br-cctv)


4. Buat Routes nya : ( IP - Routes - tambah - dst adress 10.10.20.0/24 gateway 10.10.10.2 ) tambah satu lagi 10.10.30.0/24 gateway 10.10.10.3




Sisi Client masing- masing kota 
1. buat SSTP = dial out : connect to : ip public nya - user : (sesuai vpn) password nya : (sesuai vpn) -> sesuai dibuat screet - ok

3. buat bridge

4. tambahkan port bridge : (interface: eopip-cctv dan eth3 yang masuk ke dvr nya)

5. buat dhcp client dan ip - firewall - natt - masquarede dan output eth1

6. tambahkan ip adress untuk lan lokal khusus server yang akan menampilkan cctv

7. tambahkan Dchp server (harus buat IP pool dulu ) - network - dan IP pool

8. tambahkan IP route - dst adress 10.10.20.0/24 gateway 10.10.10.1 (jika mau tembak IP yg di tuju adalah 10.10.20.1)











⚙️ BAGIAN 1 – Setting di CHR (VPN Server)

1️⃣ Aktifkan L2TP Server

Masuk ke PPP → Interface → L2TP Server

/interface l2tp-server server set enabled=yes use-ipsec=no


---

2️⃣ Buat IP Address Lokal VPN

/ip address add address=10.10.40.1/24 interface=ether1 comment="VPN local"


---

3️⃣ Buat Profile VPN

/ppp profile add name=vpn-profile local-address=10.10.10.1 remote-address=10.10.10.2 bridge=none use-encryption=yes


---

4️⃣ Buat IP Pool (untuk Client VPN)

/ip pool add name=10.10.10-pool ranges=10.10.10.2-10.10.10.10


---

5️⃣ Buat Secret untuk Client 1 & 2

/ppp secret add name=client1 password=123 service=l2tp profile=vpn-profile remote-address=10.10.10.2
/ppp secret add name=client2 password=123 service=l2tp profile=vpn-profile remote-address=10.10.10.3


---

6️⃣ (Opsional) NAT agar VPN client bisa Internet

/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade

> ether1 = interface internet CHR kamu.




---

7️⃣ Route otomatis untuk tiap client akan muncul begitu mereka connect.

Tapi jika ingin manual, tambahkan:

/ip route add dst-address=10.10.20.0/24 gateway=10.10.10.2
/ip route add dst-address=10.10.30.0/24 gateway=10.10.10.3


---

💻 BAGIAN 2 – Setting di CLIENT 1

1️⃣ Buat koneksi L2TP ke CHR

/interface l2tp-client add connect-to=<IP-PUBLIC-CHR> name=l2tp-to-chr user=client1 password=123 use-ipsec=no add-default-route=no


---

2️⃣ Tambahkan IP lokal (LAN)

/ip address add address=10.10.20.1/24 interface=bridge-local


---

3️⃣ Routing ke Client 2 lewat VPN

/ip route add dst-address=10.10.30.0/24 gateway=10.10.10.1

> Artinya: kalau mau ke jaringan Client 2, lewat VPN (gateway = CHR).




---

4️⃣ NAT untuk Internet (jika local LAN butuh internet)

/ip firewall nat add chain=srcnat out-interface=l2tp-to-chr action=masquerade


---

5️⃣ Bridge LAN (jika pakai beberapa port)

/interface bridge add name=bridge-local
/interface bridge port add bridge=bridge-local interface=ether2
/interface bridge port add bridge=bridge-local interface=ether3


---

💻 BAGIAN 3 – Setting di CLIENT 2

1️⃣ Buat koneksi L2TP ke CHR

/interface l2tp-client add connect-to=<IP-PUBLIC-CHR> name=l2tp-to-chr user=client2 password=123 use-ipsec=no add-default-route=no


---

2️⃣ Tambahkan IP lokal (LAN)

/ip address add address=10.10.30.1/24 interface=bridge-local


---

3️⃣ Routing ke Client 1 lewat VPN

/ip route add dst-address=10.10.20.0/24 gateway=10.10.10.1


---

4️⃣ NAT untuk Internet (jika local LAN butuh internet)

/ip firewall nat add chain=srcnat out-interface=l2tp-to-chr action=masquerade


---

5️⃣ Bridge LAN

/interface bridge add name=bridge-local
/interface bridge port add bridge=bridge-local interface=ether2
/interface bridge port add bridge=bridge-local interface=ether3



