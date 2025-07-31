
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


# PowerShell Automation Scripts for Microsoft 365 and Active Directory

This document contains production-ready PowerShell scripts for:

1. **Mailbox Audits**
2. **Stale Account Cleanup**
3. **License Revocation**
4. **Group Membership Syncing**

---

## 1. Mailbox Audits – Exchange Online

```powershell
<#
.SYNOPSIS
    Audit all mailboxes for:
    - Mailbox size
    - Last logon time
    - Forwarding rules

.NOTES
    Requires: Connect-ExchangeOnline module
#>

# Connect to Exchange Online
Connect-ExchangeOnline -UserPrincipalName admin@yourdomain.com

# Output file
$ReportFile = "C:\Reports\Mailbox_Audit_$(Get-Date -Format yyyy-MM-dd).csv"

# Get mailbox details
$Mailboxes = Get-Mailbox -ResultSize Unlimited

$Report = foreach ($mbx in $Mailboxes) {
    $Stats = Get-MailboxStatistics $mbx.Identity
    $Fwd = (Get-InboxRule -Mailbox $mbx.Identity -ErrorAction SilentlyContinue | Where-Object { $_.ForwardTo } | Select-Object -ExpandProperty ForwardTo) -join ", "

    [PSCustomObject]@{
        DisplayName      = $mbx.DisplayName
        PrimarySMTP      = $mbx.PrimarySmtpAddress
        LastLogonTime    = $Stats.LastLogonTime
        MailboxSizeMB    = [math]::Round($Stats.TotalItemSize.Value.ToMB(),2)
        ForwardingTo     = $Fwd
    }
}

# Save CSV report
$Report | Export-Csv -NoTypeInformation -Path $ReportFile

Write-Host "Mailbox Audit Report saved to $ReportFile" -ForegroundColor Green
```

---

## 2. Stale Account Cleanup – Active Directory

```powershell
<#
.SYNOPSIS
    Find and disable AD accounts inactive for more than 90 days.

.NOTES
    Requires RSAT ActiveDirectory module
#>

Import-Module ActiveDirectory

$DaysInactive = 90
$DateLimit = (Get-Date).AddDays(-$DaysInactive)

$StaleAccounts = Get-ADUser -Filter { Enabled -eq $true -and LastLogonDate -lt $DateLimit } -Properties LastLogonDate

foreach ($User in $StaleAccounts) {
    # Disable the account
    Disable-ADAccount -Identity $User.SamAccountName
    # Move to Disabled Users OU (Optional)
    # Move-ADObject -Identity $User.DistinguishedName -TargetPath "OU=Disabled Users,DC=yourdomain,DC=com"
    Write-Host "Disabled: $($User.SamAccountName) Last Logon: $($User.LastLogonDate)"
}

Write-Host "Total stale accounts disabled: $($StaleAccounts.Count)" -ForegroundColor Yellow
```

---

## 3. License Revocation – Microsoft 365

```powershell
<#
.SYNOPSIS
    Remove Microsoft 365 licenses from disabled accounts.

.NOTES
    Requires MSOnline module
#>

# Connect to M365
Connect-MsolService

# Get disabled users with licenses
$DisabledLicensedUsers = Get-MsolUser -All | Where-Object { $_.IsLicensed -eq $true -and $_.BlockCredential -eq $true }

foreach ($User in $DisabledLicensedUsers) {
    Set-MsolUserLicense -UserPrincipalName $User.UserPrincipalName -RemoveLicenses ($User.Licenses.AccountSkuId -join ",")
    Write-Host "Revoked licenses from: $($User.UserPrincipalName)"
}
```

---

## 4. Group Membership Syncing – AD to Microsoft 365

```powershell
<#
.SYNOPSIS
    Sync AD group membership to Microsoft 365 Security group.

.NOTES
    Requires AzureAD module (for cloud groups) + RSAT ActiveDirectory module
#>

Import-Module ActiveDirectory
Connect-AzureAD

# Local AD group & target M365 group
$LocalGroup = "AD-Sales"
$CloudGroup = "Sales-Security"

# Get AD members
$ADMembers = Get-ADGroupMember -Identity $LocalGroup -Recursive | Where-Object { $_.ObjectClass -eq "User" }

# Get Azure AD group object
$CloudGroupObj = Get-AzureADGroup -SearchString $CloudGroup
$CloudMembers = Get-AzureADGroupMember -ObjectId $CloudGroupObj.ObjectId | Select-Object -ExpandProperty UserPrincipalName

foreach ($User in $ADMembers) {
    $UPN = (Get-ADUser -Identity $User.SamAccountName).UserPrincipalName
    if ($CloudMembers -notcontains $UPN) {
        Add-AzureADGroupMember -ObjectId $CloudGroupObj.ObjectId -RefObjectId (Get-AzureADUser -ObjectId $UPN).ObjectId
        Write-Host "Added $UPN to $CloudGroup"
    }
}

Write-Host "Group sync complete."
```

---

## Scheduling Automation

You can run these scripts automatically using:

- **Task Scheduler** (on-premise server)
- **Azure Automation Account** (cloud-based, with Run As accounts)

Recommended schedule:

| Script                  | Frequency |
|-------------------------|-----------|
| Mailbox Audits          | Weekly    |
| Stale Account Cleanup   | Monthly   |
| License Revocation      | Weekly    |
| Group Membership Sync   | Daily     |




