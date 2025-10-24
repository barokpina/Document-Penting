


## ⚙️ LANGKAH 1 — Pastikan Koneksi Dasar CHR Beres

Pastikan CHR kamu:

Sudah punya IP Public (misal: 103.xx.xx.xx)

Sudah bisa internetan

Cek dengan:

```
ping 8.8.8.8
```

Pastikan tidak ada firewall yang memblokir port VPN (UDP 500, 1701, 4500)


## ⚙️ LANGKAH 2 — Buat IP Pool untuk VPN Client

Masuk ke terminal Mikrotik kamu (CHR pusat), lalu jalankan:
```
/ip pool add name=l2tp-pool ranges=10.10.10.2-10.10.10.100
```

➡️ Pool ini nanti dipakai untuk membagikan IP ke client yang connect VPN.


## ⚙️ LANGKAH 3 — Buat PPP Profile untuk VPN

Masukkan perintah ini:
```
/ppp profile add name=l2tp-profile local-address=10.10.10.1 remote-address=l2tp-pool use-encryption=yes
```

local-address=10.10.10.1 → IP server VPN

remote-address=l2tp-pool → IP client VPN


## ⚙️ LANGKAH 4 — Aktifkan L2TP Server

* Masuk menu GUI:

* PPP → Interface → L2TP Server

* Centang Enable L2TP Server

* Centang Use IPsec

* Isi IPsec Secret misalnya: vpn123

* Default Profile → pilih l2tp-profile

* Authentication: centang mschap1 dan mschap2

* Klik Apply dan OK

Atau lewat terminal:
```
/interface l2tp-server server set enabled=yes ipsec-secret=vpn123 default-profile=l2tp-profile use-ipsec=yes authentication=mschap1,mschap2

```

## ⚙️ LANGKAH 5 — Tambah User VPN (L2TP Secrets)

Untuk tiap client nanti (Bandung, Makassar, Bekasi, dsb), buat akun:
```
/ppp secret add name=clientA password=12345 service=l2tp profile=l2tp-profile
/ppp secret add name=clientB password=12345 service=l2tp profile=l2tp-profile
/ppp secret add name=clientC password=12345 service=l2tp profile=l2tp-profile
```

## ⚙️ LANGKAH 6 — Tambah NAT agar Client Bisa Akses Jaringan Internet/Lokal
```
/ip firewall nat add chain=srcnat src-address=10.10.10.0/24 action=masquerade
```

## ⚙️ LANGKAH 7 — Pastikan Port VPN Dibuka

Pastikan port berikut tidak diblokir:

UDP 500 (IPsec)

UDP 1701 (L2TP)

UDP 4500 (NAT-T)

Cek di Mikrotik:
```1
/ip firewall filter print
```

Jika ada rule yang block input, tambahkan exception:
```
/ip firewall filter add chain=input protocol=udp port=500,1701,4500 action=accept comment="Allow L2TP/IPsec"
/ip firewall filter add chain=input protocol=ipsec-esp action=accept comment="Allow IPsec"
/ip firewall filter add chain=input src-address=0.0.0.0/0 action=accept comment="Allow VPN
```





### SETTING DI CLIENT

⚙️ LANGKAH SETUP CLIENT 102-UTAMA
🧩 1️⃣ Cek interface dan IP lokal

Pastikan interface kamu seperti ini:

ether1 → internet (dari modem)
ether2–ether4 → LAN/DVR, dijadikan bridge


Buat bridge:

/interface bridge add name=BRIDGE-LAN
/interface bridge port add bridge=BRIDGE-LAN interface=ether2
/interface bridge port add bridge=BRIDGE-LAN interface=ether3
/interface bridge port add bridge=BRIDGE-LAN interface=ether4


Beri IP untuk LAN:

/ip address add address=172.168.10.1/24 interface=BRIDGE-LAN comment="LAN DVR"

🌐 2️⃣ Pastikan internet aktif di ether1

Tes koneksi internet:

/tool ping 8.8.8.8


Kalau bisa reply, lanjut.

🔗 3️⃣ Buat koneksi L2TP ke CHR
/interface l2tp-client add name=l2tp-out1 connect-to=91.192.81.98 user="102-UTAMA" password="passwordkamu" use-ipsec=yes ipsec-secret="secretkamu" profile=default keepalive-timeout=10 add-default-route=no


Catatan:

connect-to → isi IP publik CHR VPS kamu

user dan password sesuai /ppp secret di CHR

add-default-route=no supaya routing lokal tidak bentrok dengan gateway ISP

📡 4️⃣ Tambahkan IP di L2TP

Biasanya CHR akan kasih IP otomatis.
Kamu bisa cek dengan:

/ip address print


harus muncul seperti ini:

10.10.10.3/32  → interface l2tp-out1


Kalau belum muncul, set manual:

/ip address add address=10.10.10.3/32 interface=l2tp-out1

🚏 5️⃣ Tambahkan routing untuk jaringan antar client

Agar 102-UTAMA bisa akses client lain lewat CHR:

/ip route add dst-address=10.10.10.0/24 gateway=10.10.10.1 comment="Route ke semua client via CHR"


10.10.10.1 adalah IP CHR di interface L2TP (server side).

🔄 6️⃣ Aktifkan NAT agar LAN bisa internetan
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade comment="Internet NAT"

🌉 7️⃣ Aktifkan NAT agar LAN bisa akses jaringan VPN (opsional tapi penting)

Supaya LAN di 102-UTAMA (172.168.10.x) bisa akses jaringan antar client di 10.10.10.x:

/ip firewall nat add chain=srcnat out-interface=l2tp-out1 action=masquerade comment="VPN NAT"

🧠 8️⃣ Uji koneksi antar client

Dari 102-UTAMA, ping CHR:

/ping 10.10.10.1


Ping client lain, misal A87:

/ping 10.10.10.4


Kalau berhasil, lanjut cek akses ke LAN client lain nanti.

⚙️ 9️⃣ Pastikan firewall tidak memblokir

Tambahkan rule sederhana agar semua traffic VPN bisa lewat:

/ip firewall filter add chain=forward action=accept in-interface=l2tp-out1 comment="Allow VPN IN"
/ip firewall filter add chain=forward action=accept out-interface=l2tp-out1 comment="Allow VPN OUT"


✅ HASIL AKHIR:

102-UTAMA bisa internetan

102-UTAMA bisa akses CHR (10.10.10.1)

102-UTAMA bisa akses client lain (misal 10.10.10.4)

LAN (172.168.10.x) bisa akses juga ke jaringan 10.10.10.x

Kalau kamu kirim hasil dari perintah berikut di 102-UTAMA:

/ip address print
/ip route print
/interface l2tp-client print
/ip firewall nat print


saya bantu pastikan semuanya sudah benar dan bisa langsung tes antar client.

