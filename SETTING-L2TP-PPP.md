


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
