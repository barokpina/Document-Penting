SETTING SISI CLIENT 
A. SETTING AGAR BISA INTERNET DAN DI BRIGDE LAN NYA
  1. buatkan bridge lan 2 sampai 4 gabungkan (klik menu bridge → klik tab bridge → klik tombol + → name: bridge-lan → masukkan ether2 ke bridge → klik tab ports → interface: ether2 → bridge: bridge-lan)
  2. tambahkan ip - dhcp client - pastikan bound
  3. tambahkan ip address 192.168.20.1/24 di interfaces bridge-lan
  4. tambahkan ip pool name: dhcp-pool-lanaddresses: 192.168.20.100-192.168.20.200
  5. tambahkan dhcp server klik tab networks address: 192.168.20.0/24 gateway: 192.168.20.1 netmask: 24 dns servers: 8.8.8.8, 8.8.4.4
  6. tambahkan dhcp server (klik tombol +) name: dhcp-lan interface: bridge-lan lease time: 1d 00:00:00 (1 hari) address pool: dhcp-pool-lan
  7 tambahkan ip firewall - nat chain: srcnat out. interface: ether1 tab action → action: masquerade
  8. tambahkan dns ip → dns servers: 8.8.8.8, 8.8.4.4 allow remote requests: ✅ (centang)
        
B. SETTING AGAR BISA TERHUBUNG KE SERVER CHR 
   1. TAMBAHKAN L2TP CLIENT DI TAB PPP → Klik menu PPP → Klik tab Interface → Klik tombol → pilih L2TP Client  → Tab General  → Name: l2tp-to-chr → Tab Dial Out → Connect To: 91.192.81.98 (IP publik CHR server) → User: [username Anda] (contoh: 102-utama, A872, client-3)  → Password: [password Anda] 1125toki → Use IPsec: ✅ (centang)  → IPsec Secret: 1125toki (harus sama dengan server) → Add Default Route: no (jangan centang)              
     
   2. Enable Logging (untuk troubleshooting) : Klik menu System → Logging Klik tombol → Isi: Topics: centang l2tp, ipsec, ppp, infoAction: memory Klik OK
    
   3. Firewall Allow L2TP/IPsec →
      Rule 1: Allow UDP Port L2TP/IPsec Klik menu IP → Firewall Klik tab Filter Rules Klik tombol → Tab General: Chain: input Protocol: udp Dst. Port: 500,4500,1701 Tab Action: Action: accept Tab Comment: Comment: Allow L2TP/IPsec Klik OK
    
        Rule 2: Allow IPsec ESP Protocol Klik tombol → Tab General: Chain: input Protocol: ipsec-esp → Tab Action: → Action: accept Tab Comment: Comment: Allow IPsec Klik OK

