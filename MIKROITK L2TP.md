# Panduan Konfigurasi L2TP VPN - 3 MikroTik

## Topologi Jaringan

```
Internet
    |
[MikroTik CHR - Server]
IP Publik: 91.192.81.98
    |
    +--- L2TP VPN ---+
    |                |
[102-utama]      [A872]
(Client 1)       (Client 2)
Lantai 1         Lantai 3
```

---

# BAGIAN 1: KONFIGURASI CHR SERVER

## Langkah 1: Buat PPP Profile

1. Buka Winbox, connect ke MikroTik CHR (91.192.81.98)
2. Klik menu **PPP**
3. Klik tab **Profiles**
4. Klik tombol **+** (tambah)
5. Isi konfigurasi:
   - **Name**: `l2tp-profile`
   - **Use Encryption**: ✅ (centang)
6. Klik **OK**

---

## Langkah 2: Enable L2TP Server

1. Masih di menu **PPP**
2. Klik tab **Interface**
3. Klik tombol **L2TP Server** (atau klik **Interface** → **L2TP Server**)
4. Centang **Enabled**
5. Isi konfigurasi:
   - **Default Profile**: `l2tp-profile`
   - **Authentication**: `mschap2` (centang)
   - **Use IPsec**: ✅ (centang)
   - **IPsec Secret**: `RahasiaIPsec123`
   - **Keepalive Timeout**: `60`
6. Klik **OK**

---

## Langkah 3: Buat User untuk Client 1 (102-utama)

1. Klik menu **PPP**
2. Klik tab **Secrets**
3. Klik tombol **+** (tambah)
4. Isi konfigurasi:
   - **Name**: `102-utama`
   - **Password**: `Pass102utama`
   - **Service**: `l2tp` (pilih dari dropdown)
   - **Profile**: `l2tp-profile`
   - **Local Address**: `10.10.10.1`
   - **Remote Address**: `10.10.10.10`
5. Klik **OK**

---

## Langkah 4: Buat User untuk Client 2 (A872)

1. Masih di tab **Secrets**
2. Klik tombol **+** (tambah)
3. Isi konfigurasi:
   - **Name**: `A872`
   - **Password**: `PassA872`
   - **Service**: `l2tp`
   - **Profile**: `l2tp-profile`
   - **Local Address**: `10.10.10.1`
   - **Remote Address**: `10.10.10.11`
4. Klik **OK**

---

## Langkah 5: Konfigurasi Firewall CHR

### 5.1 Allow L2TP/IPsec Port

1. Klik menu **IP** → **Firewall**
2. Klik tab **Filter Rules**
3. Klik tombol **+** (tambah rule baru)
4. **Tab General:**
   - **Chain**: `input`
   - **Protocol**: `udp`
   - **Dst. Port**: `500,4500,1701`
5. **Tab Action:**
   - **Action**: `accept`
6. **Tab Comment:**
   - **Comment**: `Allow L2TP/IPsec`
7. Klik **OK**

### 5.2 Allow IPsec ESP

1. Klik tombol **+** (tambah rule baru)
2. **Tab General:**
   - **Chain**: `input`
   - **Protocol**: `ipsec-esp`
3. **Tab Action:**
   - **Action**: `accept`
4. **Tab Comment:**
   - **Comment**: `Allow IPsec ESP`
5. Klik **OK**

### 5.3 Allow VPN Client-to-Client Communication (PENTING!)

1. Klik tombol **+** (tambah rule baru)
2. **Tab General:**
   - **Chain**: `forward`
   - **Src. Address**: `10.10.10.0/24`
   - **Dst. Address**: `10.10.10.0/24`
3. **Tab Action:**
   - **Action**: `accept`
4. **Tab Comment:**
   - **Comment**: `Allow VPN inter-client`
5. Klik **OK**

---

## Langkah 6: Enable Logging (untuk Troubleshooting)

1. Klik menu **System** → **Logging**
2. Klik tombol **+** (tambah)
3. Isi konfigurasi:
   - **Topics**: centang `l2tp` dan `info`
   - **Action**: `memory`
4. Klik **OK**

5. Klik tombol **+** lagi (untuk IPsec)
6. Isi konfigurasi:
   - **Topics**: centang `ipsec` dan `info`
   - **Action**: `memory`
7. Klik **OK**

---

## Langkah 7: Setup Routing di CHR

### 7.1 Route ke Network 102-utama

1. Klik menu **IP** → **Routes**
2. Klik tombol **+**
3. Isi konfigurasi:
   - **Dst. Address**: `192.168.20.0/24`
   - **Gateway**: `10.10.10.10`
   - **Comment**: `to 102-utama LAN`
4. Klik **OK**

### 7.2 Route ke Network A872

1. Klik tombol **+**
2. Isi konfigurasi:
   - **Dst. Address**: `192.168.10.0/24`
   - **Gateway**: `10.10.10.11`
   - **Comment**: `to A872 LAN`
3. Klik **OK**

---

# BAGIAN 2: KONFIGURASI CLIENT 1 (102-UTAMA)

## Langkah 1: Setup Internet/WAN

### Jika Pakai DHCP dari Router/Modem:

1. Buka Winbox, connect ke MikroTik 102-utama
2. Klik menu **IP** → **DHCP Client**
3. Klik tombol **+**
4. Isi konfigurasi:
   - **Interface**: `ether1`
   - **Add Default Route**: `yes`
5. Klik **OK**

### Jika Pakai PPPoE:

1. Klik menu **Interfaces**
2. Klik tombol **+** → pilih **PPPoE Client**
3. Isi konfigurasi:
   - **Name**: `pppoe-out1`
   - **Interfaces**: `ether1`
   - **User**: (username dari ISP)
   - **Password**: (password dari ISP)
   - **Add Default Route**: `yes`
4. Klik **OK**

---

## Langkah 2: Setup Bridge untuk LAN (ether2, 3, 4)

### 2.1 Buat Bridge

1. Klik menu **Bridge**
2. Klik tab **Bridge**
3. Klik tombol **+**
4. Isi:
   - **Name**: `bridge-lan`
5. Klik **OK**

### 2.2 Masukkan ether2 ke Bridge

1. Klik tab **Ports**
2. Klik tombol **+**
3. Isi:
   - **Interface**: `ether2`
   - **Bridge**: `bridge-lan`
4. Klik **OK**

### 2.3 Masukkan ether3 ke Bridge

1. Klik tombol **+**
2. Isi:
   - **Interface**: `ether3`
   - **Bridge**: `bridge-lan`
3. Klik **OK**

### 2.4 Masukkan ether4 ke Bridge

1. Klik tombol **+**
2. Isi:
   - **Interface**: `ether4`
   - **Bridge**: `bridge-lan`
3. Klik **OK**

---

## Langkah 3: Set IP Address untuk Bridge LAN

1. Klik menu **IP** → **Addresses**
2. Klik tombol **+**
3. Isi:
   - **Address**: `192.168.20.1/24`
   - **Interface**: `bridge-lan`
4. Klik **OK**

---

## Langkah 4: Setup DHCP Server untuk LAN

### 4.1 Buat IP Pool

1. Klik menu **IP** → **Pool**
2. Klik tombol **+**
3. Isi:
   - **Name**: `dhcp-pool-lan`
   - **Addresses**: `192.168.20.100-192.168.20.200`
4. Klik **OK**

### 4.2 Setup DHCP Network

1. Klik menu **IP** → **DHCP Server**
2. Klik tab **Networks**
3. Klik tombol **+**
4. Isi:
   - **Address**: `192.168.20.0/24`
   - **Gateway**: `192.168.20.1`
   - **DNS Servers**: `8.8.8.8`
5. Klik **OK**

### 4.3 Buat DHCP Server

1. Klik tab **DHCP**
2. Klik tombol **+**
3. Isi:
   - **Name**: `dhcp-lan`
   - **Interface**: `bridge-lan`
   - **Address Pool**: `dhcp-pool-lan`
4. Klik **OK**

---

## Langkah 5: Setup NAT Masquerade

1. Klik menu **IP** → **Firewall**
2. Klik tab **NAT**
3. Klik tombol **+**
4. **Tab General:**
   - **Chain**: `srcnat`
   - **Out. Interface**: `ether1` (atau `pppoe-out1` jika pakai PPPoE)
5. **Tab Action:**
   - **Action**: `masquerade`
6. Klik **OK**

---

## Langkah 6: Setup DNS

1. Klik menu **IP** → **DNS**
2. Isi:
   - **Servers**: `8.8.8.8, 8.8.4.4`
   - **Allow Remote Requests**: ✅ (centang)
3. Klik **OK**

---

## Langkah 7: Enable Logging

1. Klik menu **System** → **Logging**
2. Klik tombol **+**
3. Centang: `l2tp`, `ipsec`, `ppp`, `info`
4. **Action**: `memory`
5. Klik **OK**

---

## Langkah 8: Firewall untuk L2TP Client

### 8.1 Allow L2TP/IPsec

1. Klik menu **IP** → **Firewall**
2. Klik tab **Filter Rules**
3. Klik tombol **+**
4. **Tab General:**
   - **Chain**: `input`
   - **Protocol**: `udp`
   - **Dst. Port**: `500,4500,1701`
5. **Tab Action:**
   - **Action**: `accept`
6. Klik **OK**

### 8.2 Allow IPsec ESP

1. Klik tombol **+**
2. **Tab General:**
   - **Chain**: `input`
   - **Protocol**: `ipsec-esp`
3. **Tab Action:**
   - **Action**: `accept`
4. Klik **OK**

### 8.3 Allow Traffic dari VPN Tunnel

1. Klik tombol **+**
2. **Tab General:**
   - **Chain**: `input`
   - **In. Interface**: `l2tp-to-chr` (isi manual, interface belum ada)
3. **Tab Action:**
   - **Action**: `accept`
4. **Tab Comment:**
   - **Comment**: `Allow from VPN tunnel`
5. Klik **OK**

---

## Langkah 9: Buat L2TP Client Interface

1. Klik menu **PPP**
2. Klik tab **Interface**
3. Klik tombol **+** → pilih **L2TP Client**
4. **Tab General:**
   - **Name**: `l2tp-to-chr`
   - **Connect To**: `91.192.81.98`
   - **User**: `102-utama`
   - **Password**: `Pass102utama`
5. **Tab Dial Out:**
   - **Use IPsec**: ✅ (centang)
   - **IPsec Secret**: `RahasiaIPsec123`
   - **Add Default Route**: ❌ (jangan centang)
6. **Tab Advanced:**
   - **Keepalive Timeout**: `60`
   - **Max MTU**: `1400`
   - **Max MRU**: `1400`
7. Klik **OK**

---

## Langkah 10: Cek Status Koneksi L2TP

1. Tunggu 10-30 detik
2. Di menu **PPP** → tab **Interface**
3. Lihat interface `l2tp-to-chr`, pastikan ada flag **R** (running)
4. Klik 2x pada interface tersebut, lihat **Status** harus **connected**

---

## Langkah 11: Cek IP VPN yang Didapat

1. Klik menu **IP** → **Addresses**
2. Cari interface `l2tp-to-chr`
3. Pastikan IP address: `10.10.10.10/32`

---

## Langkah 12: Test Koneksi

1. Klik menu **New Terminal**
2. Ketik: `ping 8.8.8.8 count=5` (test internet)
3. Ketik: `ping 10.10.10.1 count=5` (test VPN ke CHR)
4. Jika berhasil, lanjut ke langkah berikutnya

---

## Langkah 13: Setup Routing ke A872

1. Klik menu **IP** → **Routes**
2. Klik tombol **+**
3. Isi:
   - **Dst. Address**: `192.168.10.0/24`
   - **Gateway**: `10.10.10.1`
   - **Comment**: `to A872 via CHR`
4. Klik **OK**

---

# BAGIAN 3: KONFIGURASI CLIENT 2 (A872)

## Langkah 1: Setup Internet/WAN

### Jika Pakai DHCP dari Modem:

1. Buka Winbox, connect ke MikroTik A872
2. Klik menu **IP** → **DHCP Client**
3. Klik tombol **+**
4. Isi:
   - **Interface**: `ether1`
   - **Add Default Route**: `yes`
5. Klik **OK**

---

## Langkah 2: Setup Bridge untuk LAN (ether2, 3, 4)

### 2.1 Buat Bridge

1. Klik menu **Bridge**
2. Klik tab **Bridge**
3. Klik tombol **+**
4. Isi:
   - **Name**: `bridge-lan`
5. Klik **OK**

### 2.2 Masukkan ether2, 3, 4 ke Bridge

(Ulangi 3x untuk ether2, ether3, ether4)

1. Klik tab **Ports**
2. Klik tombol **+**
3. Isi:
   - **Interface**: `ether2` (lalu `ether3`, lalu `ether4`)
   - **Bridge**: `bridge-lan`
4. Klik **OK**

---

## Langkah 3: Set IP Address untuk Bridge LAN

1. Klik menu **IP** → **Addresses**
2. Klik tombol **+**
3. Isi:
   - **Address**: `192.168.10.1/24`
   - **Interface**: `bridge-lan`
4. Klik **OK**

---

## Langkah 4: Setup DHCP Server untuk LAN

### 4.1 Buat IP Pool

1. Klik menu **IP** → **Pool**
2. Klik tombol **+**
3. Isi:
   - **Name**: `dhcp-pool-lan`
   - **Addresses**: `192.168.10.100-192.168.10.200`
4. Klik **OK**

### 4.2 Setup DHCP Network

1. Klik menu **IP** → **DHCP Server**
2. Klik tab **Networks**
3. Klik tombol **+**
4. Isi:
   - **Address**: `192.168.10.0/24`
   - **Gateway**: `192.168.10.1`
   - **DNS Servers**: `8.8.8.8`
5. Klik **OK**

### 4.3 Buat DHCP Server

1. Klik tab **DHCP**
2. Klik tombol **+**
3. Isi:
   - **Name**: `dhcp-lan`
   - **Interface**: `bridge-lan`
   - **Address Pool**: `dhcp-pool-lan`
4. Klik **OK**

---

## Langkah 5: Setup NAT Masquerade

1. Klik menu **IP** → **Firewall**
2. Klik tab **NAT**
3. Klik tombol **+**
4. **Tab General:**
   - **Chain**: `srcnat`
   - **Out. Interface**: `ether1`
5. **Tab Action:**
   - **Action**: `masquerade`
6. Klik **OK**

---

## Langkah 6: Setup DNS

1. Klik menu **IP** → **DNS**
2. Isi:
   - **Servers**: `8.8.8.8, 8.8.4.4`
   - **Allow Remote Requests**: ✅ (centang)
3. Klik **OK**

---

## Langkah 7: Enable Logging

1. Klik menu **System** → **Logging**
2. Klik tombol **+**
3. Centang: `l2tp`, `ipsec`, `ppp`, `info`
4. **Action**: `memory`
5. Klik **OK**

---

## Langkah 8: Firewall untuk L2TP Client

### 8.1 Allow L2TP/IPsec

1. Klik menu **IP** → **Firewall**
2. Klik tab **Filter Rules**
3. Klik tombol **+**
4. **Tab General:**
   - **Chain**: `input`
   - **Protocol**: `udp`
   - **Dst. Port**: `500,4500,1701`
5. **Tab Action:**
   - **Action**: `accept`
6. Klik **OK**

### 8.2 Allow IPsec ESP

1. Klik tombol **+**
2. **Tab General:**
   - **Chain**: `input`
   - **Protocol**: `ipsec-esp`
3. **Tab Action:**
   - **Action**: `accept`
4. Klik **OK**

### 8.3 Allow dari VPN Tunnel

1. Klik tombol **+**
2. **Tab General:**
   - **Chain**: `input`
   - **In. Interface**: `l2tp-to-chr`
3. **Tab Action:**
   - **Action**: `accept`
4. Klik **OK**

### 8.4 Allow Ping dari VPN

1. Klik tombol **+**
2. **Tab General:**
   - **Chain**: `input`
   - **Src. Address**: `10.10.10.0/24`
   - **Protocol**: `icmp`
3. **Tab Action:**
   - **Action**: `accept`
4. **Tab Comment:**
   - **Comment**: `Allow ping from VPN`
5. Klik **OK**

---

## Langkah 9: Buat L2TP Client Interface

1. Klik menu **PPP**
2. Klik tab **Interface**
3. Klik tombol **+** → pilih **L2TP Client**
4. **Tab General:**
   - **Name**: `l2tp-to-chr`
   - **Connect To**: `91.192.81.98`
   - **User**: `A872`
   - **Password**: `PassA872`
5. **Tab Dial Out:**
   - **Use IPsec**: ✅ (centang)
   - **IPsec Secret**: `RahasiaIPsec123`
   - **Add Default Route**: ❌ (jangan centang)
6. **Tab Advanced:**
   - **Keepalive Timeout**: `60`
   - **Max MTU**: `1400`
   - **Max MRU**: `1400`
7. Klik **OK**

---

## Langkah 10: Cek Status Koneksi

1. Tunggu 10-30 detik
2. Di menu **PPP** → tab **Interface**
3. Lihat interface `l2tp-to-chr`, pastikan ada flag **R**
4. Klik 2x, lihat **Status** harus **connected**

---

## Langkah 11: Cek IP VPN yang Didapat

1. Klik menu **IP** → **Addresses**
2. Cari interface `l2tp-to-chr`
3. Pastikan IP: `10.10.10.11/32`

---

## Langkah 12: Test Koneksi

1. Klik menu **New Terminal**
2. Ketik: `ping 10.10.10.1 count=5` (test VPN ke CHR)
3. Jika berhasil, lanjut

---

## Langkah 13: Setup Routing ke 102-utama

1. Klik menu **IP** → **Routes**
2. Klik tombol **+**
3. Isi:
   - **Dst. Address**: `192.168.20.0/24`
   - **Gateway**: `10.10.10.1`
   - **Comment**: `to 102-utama via CHR`
4. Klik **OK**

---

# BAGIAN 4: TESTING LENGKAP

## Test dari 102-utama

1. Buka Terminal di MikroTik 102-utama
2. Ketik command berikut satu per satu:

```
ping 10.10.10.1 count=5
```
(Test ke CHR server - harus berhasil)

```
ping 10.10.10.11 count=5
```
(Test ke A872 VPN IP - harus berhasil)

```
ping 192.168.10.1 count=5
```
(Test ke A872 LAN gateway - harus berhasil)

---

## Test dari A872

1. Buka Terminal di MikroTik A872
2. Ketik command:

```
ping 10.10.10.1 count=5
```
(Test ke CHR)

```
ping 10.10.10.10 count=5
```
(Test ke 102-utama VPN IP)

```
ping 192.168.20.1 count=5
```
(Test ke 102-utama LAN gateway)

---

## Test dari CHR

1. Buka Terminal di CHR
2. Ketik:

```
ppp active print
```
(Harus muncul 2 koneksi: 102-utama dan A872)

```
ping 10.10.10.10 count=5
```
(Ping ke 102-utama)

```
ping 10.10.10.11 count=5
```
(Ping ke A872)

---

# TROUBLESHOOTING

## Jika L2TP Tidak Connect

### Cek 1: Status Interface

1. Buka menu **PPP** → tab **Interface**
2. Klik 2x pada `l2tp-to-chr`
3. Lihat **Status** - jika tidak **connected**, lanjut ke cek berikutnya

### Cek 2: Lihat Log Error

1. Klik menu **Log**
2. Cari kata: `l2tp`, `ipsec`, `error`, `timeout`
3. Screenshot atau catat error yang muncul

### Cek 3: Test Ping ke Server

1. Buka Terminal
2. Ketik: `ping 91.192.81.98 count=20`
3. Jika ada packet loss > 5%, masalah di internet client

### Cek 4: Pastikan Password & Secret Benar

1. Di client, menu **PPP** → tab **Interface**
2. Klik 2x pada `l2tp-to-chr`
3. Pastikan:
   - User: `102-utama` atau `A872`
   - Password benar
   - IPsec Secret: `RahasiaIPsec123`

---

## Jika Bisa Ping VPN tapi Tidak Bisa Ping LAN

### Cek 1: Routing Table

1. Klik menu **IP** → **Routes**
2. Pastikan ada route:
   - Di 102-utama: `192.168.10.0/24` via `10.10.10.1`
   - Di A872: `192.168.20.0/24` via `10.10.10.1`
   - Di CHR: ada 2 route ke masing-masing LAN

### Cek 2: Firewall Blocking

1. Klik menu **IP** → **Firewall** → tab **Filter Rules**
2. Pastikan tidak ada rule yang **drop** traffic dari VPN

---

## Jika Client Hilang-Timbul (Disconnect-Reconnect)

### Solusi 1: Perpanjang Keepalive

1. Di client, menu **PPP** → tab **Interface**
2. Klik 2x pada `l2tp-to-chr`
3. Tab **Advanced**
4. Ubah **Keepalive Timeout**: `120`
5. Klik **OK**

### Solusi 2: Perkecil MTU

1. Masih di interface `l2tp-to-chr`
2. Tab **Advanced**
3. Ubah:
   - **Max MTU**: `1350`
   - **Max MRU**: `1350`
4. Klik **OK**

### Solusi 3: MSS Clamping

1. Klik menu **IP** → **Firewall**
2. Klik tab **Mangle**
3. Klik tombol **+**
4. **Tab General:**
   - **Chain**: `forward`
   - **Protocol**: `tcp`
   - **TCP Flags**: centang `syn`
   - **Out. Interface**: `l2tp-to-chr`
   - **TCP MSS**: `1401-65535`
5. **Tab Action:**
   - **Action**: `change-mss`
   - **New MSS**: `1360`
6. Klik **OK**

### Solusi 4: Jika Di Belakang Router (Double NAT)

**Opsi A: DMZ Mode**
1. Login ke router di Lantai 1
2. Cari menu **DMZ** atau **Exposed Host**
3. Set DMZ IP = IP MikroTik 102-utama (dari router)
4. Enable DMZ

**Opsi B: Port Forwarding**
1. Login ke router di Lantai 1
2. Buat port forwarding:
   - UDP 500 → IP MikroTik
   - UDP 4500 → IP MikroTik
   - UDP 1701 → IP MikroTik

---

## Jika Tidak Bisa Ping Antar Client VPN

### Di CHR Server:

1. Klik menu **IP** → **Firewall** → tab **Filter Rules**
2. Pastikan ada rule:
   - **Chain**: `forward`
   - **Src. Address**: `10.10.10.0/24`
   - **Dst. Address**: `10.10.10.0/24`
   - **Action**: `accept`
3. Jika belum ada, tambahkan rule tersebut

### Di Client:

1. Klik menu **IP** → **Firewall** → tab **Filter Rules**
2. Tambahkan rule:
   - **Chain**: `input`
   - **Src. Address**: `10.10.10.0/24`
   - **Protocol**: `icmp`
   - **Action**: `accept`

---

# INFORMASI KONFIGURASI

| Parameter | Nilai |
|-----------|-------|
| **CHR IP Publik** | 91.192.81.98 |
| **CHR VPN Gateway** | 10.10.10.1 |
| **VPN Network** | 10.10.10.0/24 |
| **IPsec Secret** | RahasiaIPsec123 |
| | |
| **Client 1 (102-utama)** | |
| Username | 102-utama |
| Password | Pass102utama |
| VPN IP | 10.10.10.10 |
| LAN Network | 192.168.20.0/24 |
| LAN Gateway | 192.168.20.1 |
| Lokasi | Lantai 1 (di belakang router) |
| | |
| **Client 2 (A872)** | |
| Username | A872 |
| Password | PassA872 |
| VPN IP | 10.10.10.11 |
| LAN Network | 192.168.10.0/24 |
| LAN Gateway | 192.168.10.1 |
| Lokasi | Lantai 3 (langsung dari modem) |

---

# CATATAN PENTING

## Keamanan

⚠️ **Ganti semua password dengan yang lebih kuat!**
- Password minimal 16 karakter
- Kombinasi huruf besar, kecil, angka, simbol

## Double NAT Warning

⚠️ Client 102-utama di belakang router (double NAT) bisa menyebabkan:
- Koneksi tidak stabil
- Sering disconnect-reconnect

**Solusi terbaik:** Set router lantai 1 ke **Bridge Mode** atau **DMZ**

## Device di LAN

- **Komputer/DVR** yang colok ke ether2, 3, 4 akan otomatis dapat IP dari DHCP
- Range IP:
  - Di 102-utama: `192.168.20.100-200`
  - Di A872: `192.168.10.100-200`

## Akses DVR CCTV Antar Site

Setelah VPN terhubung, DVR CCTV bisa diakses dari site lain.

**Contoh: DVR di A872 diakses dari komputer di 102-utama**

### Set IP Static untuk DVR (Recommended)

**Opsi 1: Set di DVR langsung**
1. Masuk ke setting DVR
2. Network Settings:
   - IP Address: `192.168.10.50`
   - Subnet Mask: `255.255.255.0`
   - Gateway: `192.168.10.1`
   - DNS: `8.8.8.8`

**Opsi 2: DHCP Static Lease di MikroTik**
1. Colok DVR ke A872, biarkan dapat IP DHCP
2. Di MikroTik A872, menu **IP** → **DHCP Server** → tab **Leases**
3. Cari DVR (lihat MAC address dan hostname)
4. Klik 2x pada lease DVR
5. Klik tombol **Make Static**
6. Ubah **Address** jadi: `192.168.10.50`
7. Klik **OK**

### Akses DVR dari Site Lain

**Dari komputer di 102-utama:**
1. Buka browser
2. Ketik: `http://192.168.10.50`
3. Atau akses via aplikasi DVR dengan IP: `192.168.10.50`

**Port umum DVR:**
- HTTP: `80` atau `8000`
- RTSP: `554`
- Media Port: `34567`

---

# DIAGRAM ALUR KOMUNIKASI

```
┌─────────────────────────────────────────────────────────┐
│                    INTERNET                             │
└────────────────────┬────────────────────────────────────┘
                     │
         ┌───────────▼───────────┐
         │   CHR Server (VPS)    │
         │   IP Publik:          │
         │   91.192.81.98        │
         │   VPN Gateway:        │
         │   10.10.10.1          │
         └───────┬───────┬───────┘
                 │       │
        L2TP VPN │       │ L2TP VPN
                 │       │
    ┌────────────▼─┐   ┌─▼────────────┐
    │  102-utama   │   │    A872      │
    │  Lantai 1    │   │  Lantai 3    │
    │  (Router)    │   │  (Modem)     │
    └──────┬───────┘   └───────┬──────┘
           │                   │
           │ VPN IP:           │ VPN IP:
           │ 10.10.10.10       │ 10.10.10.11
           │                   │
    ┌──────▼───────┐    ┌──────▼───────┐
    │ Bridge-LAN   │    │ Bridge-LAN   │
    │ 192.168.20.1 │    │ 192.168.10.1 │
    │              │    │              │
    │ ether2,3,4   │    │ ether2,3,4   │
    └──┬───┬───┬───┘    └──┬───┬───┬───┘
       │   │   │           │   │   │
       ▼   ▼   ▼           ▼   ▼   ▼
      PC  PC  PC         PC  DVR PC
   .100 .101 .102      .100 .50 .101
```

## Contoh Komunikasi: PC di 102-utama → DVR di A872

```
1. PC (192.168.20.100) ingin akses DVR (192.168.10.50)
   ↓
2. Request ke gateway MikroTik 102-utama (192.168.20.1)
   ↓
3. MikroTik cek routing table:
   - Dst: 192.168.10.0/24 → Gateway: 10.10.10.1
   ↓
4. Paket dikirim via L2TP tunnel (10.10.10.10 → 10.10.10.1)
   ↓
5. CHR Server (10.10.10.1) terima paket
   - Cek routing: 192.168.10.0/24 → Gateway: 10.10.10.11
   - Firewall forward: Allow (rule VPN inter-client)
   ↓
6. Paket dikirim via L2TP tunnel ke A872 (10.10.10.11)
   ↓
7. MikroTik A872 terima paket, forward ke DVR (192.168.10.50)
   ↓
8. DVR terima request dan balas
   ↓
9. Response kembali via jalur yang sama (A872 → CHR → 102-utama → PC)
```

---

# MONITORING & MAINTENANCE

## Cek Status Koneksi VPN

### Di CHR Server:

1. Menu **PPP** → tab **Interface**
2. Lihat daftar client yang connected (harus ada flag **R**)

3. Atau menu **New Terminal**, ketik:
   ```
   ppp active print
   ```

**Output yang benar:**
```
0 R name="102-utama" service=l2tp address=10.10.10.10 uptime=1h23m45s
1 R name="A872" service=l2tp address=10.10.10.11 uptime=2h15m30s
```

### Di Client:

1. Menu **PPP** → tab **Interface**
2. Klik 2x pada `l2tp-to-chr`
3. Lihat:
   - **Status**: harus **connected**
   - **Uptime**: berapa lama sudah connected
   - **Encoding**: MPPE128 (enkripsi aktif)

---

## Monitoring Traffic VPN

### Bandwidth Test Antar Client:

1. Di MikroTik 102-utama, buka Terminal
2. Ketik:
   ```
   tool bandwidth-test 10.10.10.11 protocol=tcp duration=10s
   ```
3. Lihat hasil throughput VPN antar site

### Monitor Penggunaan Interface:

1. Menu **Interfaces**
2. Klik pada interface `l2tp-to-chr`
3. Lihat **Tx** dan **Rx** untuk melihat traffic

---

## Cek Log Secara Berkala

### Melihat Log L2TP/IPsec:

1. Menu **Log**
2. Filter dengan kata kunci:
   - `l2tp`
   - `ipsec`
   - `error`
   - `timeout`

### Melihat Log Real-time:

1. Menu **New Terminal**
2. Ketik:
   ```
   log print follow where topics~"l2tp"
   ```
3. Biarkan berjalan untuk monitoring live

---

## Backup Konfigurasi

### Backup via Winbox:

1. Menu **Files**
2. Klik **Backup**
3. Isi:
   - **Name**: `backup-chr-2025-01-01` (sesuaikan tanggal)
   - **Password**: (opsional, untuk enkripsi)
4. Klik **Backup**
5. File akan muncul di **Files**, download ke komputer

### Export Configuration (Text):

1. Menu **New Terminal**
2. Ketik:
   ```
   export file=config-chr-2025-01-01
   ```
3. Menu **Files**, download file `.rsc`

**Lakukan backup rutin untuk:**
- CHR Server (paling penting)
- 102-utama
- A872

---

# OPTIMASI PERFORMA

## Jika Koneksi VPN Lambat

### 1. Test Bandwidth Internet

**Di Client:**
1. Menu **New Terminal**
2. Test ke server internet:
   ```
   tool bandwidth-test 8.8.8.8 protocol=tcp duration=30s direction=both
   ```
3. Lihat upload dan download speed

**Jika internet lambat, VPN juga akan lambat.**

### 2. Test Latency (Ping Time)

```
ping 10.10.10.1 count=50
```

**Ideal:**
- Ping time < 50ms: Excellent
- Ping time 50-100ms: Good
- Ping time 100-200ms: Acceptable
- Ping time > 200ms: Perlu optimasi

### 3. Disable Compression (Jika CPU Tinggi)

1. Di CHR, menu **PPP** → tab **Profiles**
2. Klik 2x pada `l2tp-profile`
3. **Use Compression**: `no`
4. Klik **OK**

### 4. Gunakan Hardware Acceleration (Jika Ada)

1. Menu **System** → **Resource**
2. Lihat **CPU** usage
3. Jika CPU tinggi (>70%), pertimbangkan upgrade hardware CHR

---

# PENAMBAHAN CLIENT BARU

Jika ingin menambah client ke-3, ke-4, dst:

## Di CHR Server:

### Langkah 1: Tambah User Baru

1. Menu **PPP** → tab **Secrets**
2. Klik **+**
3. Isi:
   - **Name**: `client-3`
   - **Password**: `PassClient3`
   - **Service**: `l2tp`
   - **Profile**: `l2tp-profile`
   - **Local Address**: `10.10.10.1`
   - **Remote Address**: `10.10.10.12` (increment IP)
4. Klik **OK**

### Langkah 2: Tambah Routing

1. Menu **IP** → **Routes**
2. Klik **+**
3. Isi:
   - **Dst. Address**: `192.168.30.0/24` (network client baru)
   - **Gateway**: `10.10.10.12`
   - **Comment**: `to client-3 LAN`
4. Klik **OK**

## Di Client Baru:

Ulangi langkah konfigurasi seperti **BAGIAN 2** atau **BAGIAN 3**, dengan penyesuaian:
- Username: `client-3`
- Password: `PassClient3`
- LAN Network: `192.168.30.0/24` (pastikan tidak bentrok)
- Route ke client lain via gateway `10.10.10.1`

---

# FAQ (FREQUENTLY ASKED QUESTIONS)

## Q1: Apakah bisa akses MikroTik dari site lain?

**Jawab:** Ya, bisa!

**Contoh: Akses Winbox MikroTik A872 dari 102-utama**

1. Di MikroTik A872, pastikan firewall allow:
   - Menu **IP** → **Firewall** → **Filter Rules**
   - Tambah rule:
     - **Chain**: `input`
     - **Src. Address**: `10.10.10.0/24` atau `192.168.20.0/24`
     - **Dst. Port**: `8291` (Winbox port)
     - **Protocol**: `tcp`
     - **Action**: `accept`

2. Dari komputer di 102-utama, buka Winbox:
   - Connect to: `192.168.10.1` atau `10.10.10.11`

---

## Q2: Bagaimana cara akses DVR dari internet (bukan VPN)?

**Jawab:** Perlu **Port Forwarding** di MikroTik client.

**Contoh: DVR di A872 (192.168.10.50) ingin diakses dari internet**

1. Di MikroTik A872, menu **IP** → **Firewall** → **NAT**
2. Klik **+**
3. **Tab General:**
   - **Chain**: `dstnat`
   - **Protocol**: `tcp`
   - **Dst. Port**: `8080` (port eksternal)
   - **In. Interface**: `ether1`
4. **Tab Action:**
   - **Action**: `dst-nat`
   - **To Addresses**: `192.168.10.50`
   - **To Ports**: `80` (port DVR)
5. Klik **OK**

**Akses dari internet:** `http://[IP-Public-A872]:8080`

⚠️ **Warning:** Port forwarding membuka akses dari internet, pastikan DVR punya password kuat!

---

## Q3: Kenapa ping dari 102-utama ke A872 lambat?

**Penyebab umum:**
1. **Internet client lambat** - Test bandwidth
2. **Latency tinggi** - Cek ping time ke CHR
3. **CPU CHR tinggi** - Upgrade VPS
4. **Packet loss** - Cek koneksi internet

**Solusi:**
- Test ping langsung ke CHR dari kedua client
- Bandingkan latency
- Jika salah satu client lambat, fokus perbaiki koneksi client tersebut

---

## Q4: Apakah VPN ini aman?

**Jawab:** Ya, aman dengan enkripsi:
- **L2TP**: Tunneling protocol
- **IPsec**: Enkripsi dengan MPPE128
- **Authentication**: MSCHAP2

**Untuk keamanan lebih:**
- Ganti password default
- Gunakan password kompleks minimal 16 karakter
- Update RouterOS secara berkala
- Monitor log secara rutin

---

## Q5: Berapa bandwidth yang diperlukan untuk VPN stabil?

**Minimal:**
- **Upload**: 512 kbps per site
- **Download**: 1 Mbps per site

**Recommended untuk CCTV streaming:**
- **Upload**: 2-5 Mbps per site (tergantung jumlah kamera)
- **Download**: 5-10 Mbps per site

**Perhitungan:**
- 1 kamera 1080p @ 2Mbps
- 4 kamera = 8 Mbps upload diperlukan

---

## Q6: Apakah bisa pakai IP publik dinamis?

**Jawab:** Ya, tapi perlu DDNS (Dynamic DNS).

**Solusi:**
1. Gunakan layanan DDNS gratis:
   - No-IP
   - DynDNS
   - DuckDNS

2. Setup DDNS di CHR:
   - Menu **IP** → **Cloud**
   - Enable **DDNS Enabled**
   - Atau gunakan script untuk update DDNS provider

3. Di client, ganti **Connect To** dari IP ke hostname DDNS:
   - Contoh: `mychr.ddns.net` instead of `91.192.81.98`

---

## Q7: Bagaimana cara reset jika lupa password?

**Jika lupa password MikroTik:**

1. **Reset via Netinstall** (data hilang semua):
   - Download Netinstall dari mikrotik.com
   - Boot MikroTik via Netinstall
   - Install ulang RouterOS

2. **Reset via Physical Button** (jika ada):
   - Cabut power
   - Tekan dan tahan tombol reset
   - Colok power sambil tetap tekan reset
   - Tunggu 10 detik, lepas
   - MikroTik reset ke default

⚠️ **Setelah reset, semua konfigurasi hilang!** Harus setup ulang dari awal.

**Pencegahan:**
- Backup konfigurasi secara berkala
- Catat password di tempat aman

---

# KONTAK & SUPPORT

## Dokumentasi Official MikroTik:
- Wiki: https://wiki.mikrotik.com
- Manual: https://help.mikrotik.com
- Forum: https://forum.mikrotik.com

## Tips Troubleshooting:
1. **Selalu cek log terlebih dahulu**
2. **Test koneksi step-by-step** (internet → VPN → routing)
3. **Bandingkan dengan client yang berfungsi**
4. **Backup sebelum perubahan besar**
5. **Catat setiap perubahan yang dilakukan**

---

# CHECKLIST SETUP

Gunakan checklist ini untuk memastikan setup lengkap:

## CHR Server:
- [ ] PPP Profile dibuat
- [ ] L2TP Server enabled
- [ ] User 102-utama dibuat (IP: 10.10.10.10)
- [ ] User A872 dibuat (IP: 10.10.10.11)
- [ ] Firewall allow L2TP/IPsec
- [ ] Firewall allow VPN inter-client
- [ ] Logging enabled
- [ ] Route ke 192.168.20.0/24 via 10.10.10.10
- [ ] Route ke 192.168.10.0/24 via 10.10.10.11

## Client 102-utama:
- [ ] Internet/WAN configured (ether1)
- [ ] Bridge dibuat (bridge-lan)
- [ ] ether2, 3, 4 masuk bridge
- [ ] IP address bridge: 192.168.20.1/24
- [ ] DHCP Server active
- [ ] NAT masquerade configured
- [ ] DNS configured
- [ ] Firewall allow L2TP/IPsec
- [ ] L2TP Client created dan connected
- [ ] IP VPN: 10.10.10.10
- [ ] Route ke 192.168.10.0/24 via 10.10.10.1
- [ ] Test ping ke CHR berhasil
- [ ] Test ping ke A872 berhasil

## Client A872:
- [ ] Internet/WAN configured (ether1)
- [ ] Bridge dibuat (bridge-lan)
- [ ] ether2, 3, 4 masuk bridge
- [ ] IP address bridge: 192.168.10.1/24
- [ ] DHCP Server active
- [ ] NAT masquerade configured
- [ ] DNS configured
- [ ] Firewall allow L2TP/IPsec
- [ ] Firewall allow ping from VPN
- [ ] L2TP Client created dan connected
- [ ] IP VPN: 10.10.10.11
- [ ] Route ke 192.168.20.0/24 via 10.10.10.1
- [ ] Test ping ke CHR berhasil
- [ ] Test ping ke 102-utama berhasil

## Final Testing:
- [ ] Ping 102-utama ↔ CHR berhasil
- [ ] Ping A872 ↔ CHR berhasil
- [ ] Ping 102-utama ↔ A872 berhasil
- [ ] Ping 102-utama LAN ↔ A872 LAN berhasil
- [ ] PC di 102-utama bisa akses DVR di A872
- [ ] PC di A872 bisa akses device di 102-utama
- [ ] Koneksi stabil (tidak disconnect)
- [ ] Backup konfigurasi semua device

---

**SELESAI! VPN L2TP Site-to-Site berhasil dikonfigurasi! 🎉**

*Terakhir diupdate: 25 Oktober 2025*
*Versi dokumen: 2.0 (Step-by-Step tanpa bash)*
