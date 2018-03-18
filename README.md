$ErrorActionPreference = "SilentlyContinue"
 
# Uninstall SCCM Agent with smssetup.exe
$MyPath = $env:WinDir
& "$MyPath\ccmsetup\ccmsetup.exe" /uninstall | Out-Null
 
 
# Stop Services
Stop-Service -Name 'ccmsetup' -Force 
Stop-Service -Name 'CcmExec' -Force 
Stop-Service -Name 'smstsmgr' -Force 
Stop-Service -Name 'CmRcService' -Force 
 
# Remove Services
sc delete ccmsetup
sc delete CcmExec
sc delete smstsmgr
sc delete CmRcService
 
# Remove WMI Namespaces
Get-WmiObject -query "SELECT * FROM __Namespace WHERE Name='CCM'" -Namespace "root" | Remove-WmiObject 
Get-WmiObject -query "SELECT * FROM __Namespace WHERE Name='SMS'" -Namespace "root\cimv2"  | Remove-WmiObject 
 
# Remove Services from Registry
$MyPath = “HKLM:\SYSTEM\CurrentControlSet\Services”
Remove-Item -Path "$MyPath\CCMSetup" -Force -Recurse 
Remove-Item -Path "$MyPath\CcmExec" -Force -Recurse 
Remove-Item -Path "$MyPath\smstsmgr" -Force -Recurse
Remove-Item -Path "$MyPath\CmRcService" -Force -Recurse 
 
# Remove SCCM Client from Registry
$MyPath = “HKLM:\SOFTWARE\Microsoft”
Remove-Item -Path "$MyPath\CCM" -Force -Recurse 
Remove-Item -Path "$MyPath\CCMSetup" -Force -Recurse 
Remove-Item -Path "$MyPath\SMS" -Force -Recurse
 
# Remove SCCM Client from 64 Bit Registry
$MyPath = "HKLM:\SOFTWARE\Wow6432Node\Microsoft"
Remove-Item -Path "$MyPath\SMS" -Force -Recurse 
Remove-Item -Path "$MyPath\CCM" -Force -Recurse 
 
# Remove Folders and Files
$MyPath = $env:WinDir
Remove-Item -Path "$MyPath\CCM" -Force -Recurse
Remove-Item -Path "$MyPath\ccmsetup" -Force -Recurse
Remove-Item -Path "$MyPath\ccmcache" -Force -Recurse
Remove-Item -Path "$MyPath\SMSCFG.ini" -Force
Remove-Item -Path "$MyPath\SMS*.mif" -Force
 
# Remove Scheduled Task
Unregister-ScheduledTask -TaskName "Configuration Manager Health Evaluation" -Confirm:$False -ErrorAction SilentlyContinue
Unregister-ScheduledTask -TaskName "Configuration Manager Idle Detection" -Confirm:$False -ErrorAction SilentlyContinue
Unregister-ScheduledTask -TaskName "Configuration Manager Passport for Work Certificate Enrollment Task" -Confirm:$False -ErrorAction SilentlyContinue
 
# Remove Scheduled Task Folder
$scheduleObject = New-Object -ComObject schedule.service
$scheduleObject.connect()
$rootFolder = $scheduleObject.GetFolder("\Microsoft")
$rootFolder.DeleteFolder("Configuration Manager",$unll)
 
# Remove Certificates
Get-ChildItem -Path cert:\LocalMachine\SMS | Remove-Item
