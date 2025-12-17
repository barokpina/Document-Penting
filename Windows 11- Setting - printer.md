## Cara ini adalah : windows 11 agar bisa koneksi print ke windows 10


1. Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows NT\Printers\RPC (klik kanan - 32 bit - RpcOverTcp , RpcOverNamedPipes = nilai 1 )

2. Computer\HKEY_CURRENT_USER\Software\Microsoft\Windows NT\CurrentVersion\Windows ( klik kanan di windows - permission - klik full )

3. Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Print ( klik kanan - 32 bit - RpcAuthnLevelPrivacyEnabled )

4. windows - services - print sooler - restart 



```
@echo off
echo ======================================
echo Setting Printer RPC Registry...
echo ======================================

:: Buat key jika belum ada
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Printers\RPC" /f

:: Tambah DWORD RpcOverTcp = 1
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Printers\RPC" ^
 /v RpcOverTcp /t REG_DWORD /d 1 /f

:: Tambah DWORD RpcOverNamedPipes = 1
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Printers\RPC" ^
 /v RpcOverNamedPipes /t REG_DWORD /d 1 /f

echo.
echo Registry berhasil ditambahkan.
echo Silakan restart komputer atau service Print Spooler.
pause
```