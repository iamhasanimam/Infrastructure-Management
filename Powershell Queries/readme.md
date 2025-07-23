
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



