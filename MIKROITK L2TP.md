SETTING SISI CLIENT 
1. SETTING AGAR BISA INTERNET DAN DI BRIGDE LAN NYA
   - SETTING DHCP CLIENT DAN PASITKAN BOUND
   - BUATKAN BRIGDE LAN 2 SAMPAI 4 GABUNGKAN ( Klik menu Bridge, Klik tab Bridge Klik tombol + , Name: bridge-lan Masukkan ether2 ke Bridge Klik tab Ports Interface: ether2 Bridge: bridge-lan.
   - TAMBAHKAN IP ADDRESS 192.168.20.1/24 DI INTERFACES BRIDGE LAN
   - TAMBAHKAN IP POOL ( NAME : dhcp-pool-lan , Addresses: 192.168.20.100-192.168.20.200 )
   - TAMBAHKAN DHCP SERVER - Klik tab Networks (Address: 192.168.20.0/24 , Gateway: 192.168.20.1, Netmask: 24, DNS Servers: 8.8.8.8,8.8.4.4
   - TAMBAHKAN DHCP SERVER KLIK TOMBOL + (Name: dhcp-lan, Interface: bridge-lan , Lease Time: 1d 00:00:00 (1 hari), Address Pool: dhcp-pool-lan)
   - TAMBAHKAN IP FIREWALL - NAT (Chain: srcnat, Out. Interface: ether1 , Tab Action: Action: masquerade)
   - TAMBAHKAN DNS (IP - DNS - Servers: 8.8.8.8, 8.8.4.4 , Allow Remote Requests: ✅ (centang)


2. SETTING AGAR BISA TERHUBUNG KE SERVER CHR 
- TAMBAHKAN L2TP CLIENT DI TAB PPP (
        + Klik menu PPP
        + Klik tab Interface
        + Klik tombol + → pilih L2TP Client
        + Tab General:
        + Name: l2tp-to-chr        
        + Tab Dial Out:
        + Connect To: 91.192.81.98 (IP publik CHR server)
        + User: [username Anda] (contoh: 102-utama, A872, client-3)
        + Password: [password Anda] 1125toki
        + Use IPsec: ✅ (centang)
        + IPsec Secret: 1125toki (harus sama dengan server)
        + Add Default Route: no (jangan centang)
  )
  - Enable Logging (untuk troubleshooting)
        + Klik menu System → Logging
        + Klik tombol +
        + Isi:
        + Topics: centang l2tp, ipsec, ppp, info
        + Action: memory
        + Klik OK
    
  -  Firewall Allow L2TP/IPsec
        Rule 1: Allow UDP Port L2TP/IPsec
        + Klik menu IP → Firewall
        + Klik tab Filter Rules
        + Klik tombol +
        + Tab General:
        + Chain: input
        + Protocol: udp
        + Dst. Port: 500,4500,1701
        + Tab Action:
        + Action: accept
        + Tab Comment:
        + Comment: Allow L2TP/IPsec
        + Klik OK
          
        Rule 2: Allow IPsec ESP Protocol
        + Klik tombol +
        + Tab General:
        + Chain: input
        + Protocol: ipsec-esp
        + Tab Action:
        + Action: accept
        + Tab Comment:
        + Comment: Allow IPsec ESP
        + Klik OK

