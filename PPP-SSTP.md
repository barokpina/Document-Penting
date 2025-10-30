Sisi Server CHR nya : 
1. buat SSTP = (PPP - SSTP SERVER - enable )
2. buat scret nya = ( PPP - screts - name : kotaA - password : bebas - local adress : 172.16.1.1 -
remote adress : 172.16.1.2 ) -> buatkan juga untuk kotaB ( Remote Adress : 172.16.1.3 - sisanya sama)
3. buat EOIP Tunnel = (name : eoip-cctv-kotaA - local adress: 172.16.1.1 - remote adress: 172.16.1.2 - tunnl id: 1976) - buatkan juga untuk kotaB - dengan remote adress : sesuaikan dan Tunnel id harus beda juga.
4. buat bridge = (buka bidge - klik + dan buat nama : br-cctv)
5. Masukan port ke bridge = (klik tab port - klik + masukan interface : dari EOIP yg dibuat sebelum nya dan ether dari VPN nya )
6. buat ip adress = (IP - adresess - klik + - adresss : 192.168.3.1/24 - interface: br-cctv)


Sisi Client masing- masing kota 
1. buat SSTP = dial out : connect to : ip public nya - user : (sesuai vpn) password nya : (sesuai vpn) -> sesuai dibuat screet - ok
2. buat EOIP = (name: eoip-cctv - local adress: 172.16.1.2 remote adress: 172.16.1.1 - tunnerl id : 1976 ( samain kayak di server) - klik OK
3. buat bridge
4. tambahkan port bridge : (interface: eopip-cctv dan eth3 yang masuk ke dvr nya)
5. tambahkan ip adress untuk lan lokal khusus server yang akan menampilkan cctv (/ip route add dst-address=192.168.3.0/24 gateway=192.168.4.1
)

