HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Defender

1. DisableAntiSpyware (Set Nilai 1)
2. DisableRealtimeMonitoring (Set Nilai 1)
3. DisableRoutinelyTakingAction (Set Nilai 1)
4. DisableAntiVirus (Set Nilai 1)
5. DisableSpecialRunningModes (Set Nilai 1)
6. ServiceKeepAlive (Set Nilai 0)




File Bath nya
````
@echo off
echo Disable Windows Defender...
echo.

REG ADD "HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Defender" /v DisableAntiSpyware /t REG_DWORD /d 1 /f
REG ADD "HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Defender" /v DisableRealtimeMonitoring /t REG_DWORD /d 1 /f
REG ADD "HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Defender" /v DisableRoutinelyTakingAction /t REG_DWORD /d 1 /f
REG ADD "HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Defender" /v DisableAntiVirus /t REG_DWORD /d 1 /f
REG ADD "HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Defender" /v DisableSpecialRunningModes /t REG_DWORD /d 1 /f
REG ADD "HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Defender" /v ServiceKeepAlive /t REG_DWORD /d 0 /f

echo.
echo Selesai. Silakan restart Windows.
pause
````

