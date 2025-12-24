### Sisi Server CHR nya : 
- buat SSTP atau L2TP =
```
1. PPP - SSTP SERVER - enable (tanpa aktifkan IP sec)
```

- buat scret nya =
```
1. PPP - screts +
2. name : kotaA 
3. password : bebas 
4. local adress : 10.10.10.1 -
5. remote adress : 10.10.10.2) 
6. note : buatkan juga untuk kotaB ( Remote - Adress : 10.10.10.3- sisanya sama)
```

- buat ip adress (opsional):
```
1. IP - adresess :
2. klik + 
3. adresss : 10.10.40.1/24 
4. interface: br-cctv)
```


- Buat Routes nya : ( ini Wajib Semua Route masuk kesini)
```
1. IP - Routes - tambah : 
2. dst adress 10.10.20.0/24 
3. gateway 10.10.10.2) 
4. note : semua client masukin kesini
```


### Sisi Client masing- masing kota 
- koneksikan SSTP atau L2TP - L2TP client
1. dial out - connect to : 
2. ip public nya - user : (sesuai vpn) 
3. password nya : (sesuai vpn) -> 
4. sesuai dibuat screet - ok

- buat bridge
1. tambahkan port bridge : semua eth 2 - 5

- buat dhcp client

- ip - firewall - natt - masquarede dan output eth1
- tambahkan ip adress : 
1. IP adress : 10.10.20.1
2. interface: bridge1

- tambahkan IP pool : 
1. adress : 10.10.20.10-10.10.20.200

- tambahkan Dchp server 
1.Dhcp server - pilihan interface: IP poo
2.Network : adresss : 10.10.20.0/24 - DNS : 8.8.8. Dan 8.8.4.4

- tambahkan IP route : 
1. dst adress 10.10.20.0/24
2. gateway 10.10.10.1 
3. note : (jika mau tembak IP yg di tuju adalah 10.10.20.1)



