
### Create a PowerShell script that lists:
Network adapters IP addresses MAC addresses Default gateway Link speed Interface index (for routing)

#### individual scripts
Get-NetAdapter | Where-Object {$_.Status -eq "up"} | Select-Object Name, InterfaceDescription, MacAddress, LinkSpeed, ifIndex

Get-NetIPAddress | Where-Object {$_.AddressFamily -eq "IPv4"} | Select-Object IPAddress, InterfaceAlias, PrefixLength

Get-NetRoute -DestinationPrefix "0.0.0.0/0" | Select-Object NextHop, InterfaceAlias, RouteMetric

#### Combined script that shows on PS
Write-Host "`n ===Active Network Adapters===`n"
Get-NetAdapter | Where-Object {$_.Status -eq "up"} | Select-Object Name, InterfaceDescription, MacAddress, LinkSpeed, ifIndex

Write-Host "`n ===IP Addresses===`n"
Get-NetIPAddress | Where-Object {$_.AddressFamily -eq "IPv4"} | Select-Object IPAddress, InterfaceAlias, PrefixLength

Write-Host "`n ===Default Gateway===`n"
Get-NetRoute -DestinationPrefix "0.0.0.0/0" | Select-Object NextHop, InterfaceAlias, RouteMetric

#### Now saving it in .txt

#Get-NetworkInfo.ps1
#save full network info in a log file

New-Item -ItemType Directory -Path "C:\Users\ihasa\Desktop\Logs" -Force | Out-Null

$log = "C:\Users\ihasa\Desktop\Logs\logs.txt"

"===Active Network Adapters===" | Out-File -FilePath $log
Get-NetAdapter | Where-Object {$_.Status -eq "up"} | Out-String | Out-File -FilePath $log -Append

"===IP Addresses===`n" | Out-File -Append $log
Get-NetIPAddress | Where-Object {$_.AddressFamily -eq "IPv4"} | Out-String | Out-File -Append $log

"===Default Gateway===`n" | Out-File -Append $log
Get-NetRoute -DestinationPrefix "0.0.0.0/0" | Select-Object NextHop, InterfaceAlias, RouteMetric | Out-String | Out-File -Append $log

Write-Host "`n saved to log"

### Create a script that shows:

All stopped or disabled services

Get-WmiObject -Class win32_service | Where-Object {$_.state -ne "running"} | Select-Object Name, DisplayName, State , StartMode

#Get-servicestatus.ps1

List all service that are stopped or disabled 

$logFolder = "C:\Users\ihasa\Desktop\Logs"
$logFile = Join-Path $logFolder "service-status.txt"

"===Stopped or disabled services===" | out-file -FilePath $logFile

Get-WmiObject -Class win32_service | Where-Object {$_.state -ne "running"} | Select-Object Name, DisplayName, State , StartMode |  Out-File -Append $logFile

Write-Host "Sevice status saved to $logFile"

> We can start each service using 

```
$services = Get-Service | Where-Object { $_.Status -eq 'Stopped' }

foreach ($service in $services) {
    try {
        Start-Service -Name $service.Name -ErrorAction Stop
        Write-Host "Started: $($service.Name)"
    }
    catch {
        Write-Warning "Error starting $($service.Name): $($_.Exception.Message)"
    }
}

```



## Files modified since days

$Path = "C:\Windows"

$yesterday = (Get-Date).AddDays(-1)

Get-ChildItem -Path $Path | Where-Object {$_.LastWriteTime -gt $yesterday}


$Path = "C:\Windows"

$since = (Get-Date).AddDays(-1)

Get-ChildItem -Path $Path | Where-Object {$_.LastWriteTime -gt $since} | Select-Object Name, LastWriteTime


# Change RDP Port via PowerShell Only

This guide shows how to **change the Remote Desktop Protocol (RDP) port** entirely using **PowerShell**, without manually editing the Windows Firewall or Registry.

---

## **1️⃣ Script to Change RDP Port**

```powershell
# Change <YourPort> to desired port number
$NewPort = 3389

# 1️⃣ Remove old custom RDP firewall rules (optional cleanup)
Get-NetFirewallRule -DisplayName "RDP Custom Port*" -ErrorAction SilentlyContinue | Remove-NetFirewallRule

# 2️⃣ Change RDP registry value
Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' `
-Name "PortNumber" -Value $NewPort -Type DWord

# 3️⃣ Update firewall rule for RDP
New-NetFirewallRule -DisplayName "RDP Custom Port $NewPort" `
-Protocol TCP -LocalPort $NewPort -Direction Inbound -Action Allow

# 4️⃣ Restart Remote Desktop service so change takes effect
Stop-Service TermService -Force
Start-Service TermService

# 5️⃣ Verify RDP is listening on the new port
Get-NetTCPConnection | Where-Object { $_.LocalPort -eq $NewPort -and $_.State -eq "Listen" }
```

---

## **2️⃣ Usage**
1. Open **PowerShell as Administrator**.
2. Paste and run the script.
3. Connect via RDP using:
   ```bash
   mstsc /v:<ServerIP>:<Port>
   ```
   Example:
   ```bash
   mstsc /v:203.0.113.25:3389
   ```

---

## **3️⃣ Notes**
- **Default RDP port**: `3389`
- Valid custom ports: `1025–65535` (avoid ports already in use)
- Changing the RDP port increases **security through obscurity**, but **does not replace firewall and account security**.
- If you have a **static firewall rule for 3389**, update it or allow the new port.

---

## **4️⃣ Verify Current RDP Port**
To check what port RDP is listening on:
```powershell
Get-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' PortNumber
```
To verify if it’s active:
```powershell
Get-NetTCPConnection | Where-Object { $_.State -eq "Listen" -and $_.LocalPort -ge 1024 } | Sort-Object LocalPort
```

---




