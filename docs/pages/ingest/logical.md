---
title: Logical copy
layout: default
nav_order: 4
parent: Ingest
---

Powershell:
```
robocopy [source] "[target]\Media in\Content"  /MIR /DCOPY:T /LOG+:"[target]\Media in\Metadata\File copy log.txt" /tee
```

Powershell:
```
Write-Host "This script is designed to make a full copy of digital media attached to your machine via USB.`nThis should not be used for audio CDs (eg CDs for which you cannot see the contents as individual files)`n`nData will be copied to [Target]\[Collection code]\[Timestamped folder]\, with metadata in a parallel \Metadata\ directory`n`n"
[string]$Timestamp = Get-Date -Format yyyy_MM_dd_HHmm
[string]$drive = Read-Host -prompt "Copy from which drive? (Give single letter)"
While ($drive -notmatch ('^[a-zA-Z]$')) {[string]$drive = Read-Host -prompt "Drive to copy from must be a single letter"}
Write-Host "`n"
[string]$CollCode = Read-Host -prompt "Collection code of material to copy?"
Write-Host "`n`nThe Collection Code is $($CollCode) and the drive to copy from is $($drive):`n"
Do {$confirm = Read-Host -prompt "Do you wish to copy with these settings? (y/n)"}
While ($confirm -notmatch ('y|n|yes|no'))
if ($confirm -match ('n|no'))
	{Write-Host "`nQuitting without copying`n"
	Pause
	Return}
New-Item -ItemType Directory -path ('[Target]\' + $CollCode + '\Metadata\' + $Timestamp) -force
gci -path ${drive}: -recurse -force > ('[Target]\' + $CollCode + '\Metadata\' + $Timestamp + '\Directory listing.txt')
Get-WmiObject -Class Win32_volume -Filter "DriveLetter = '${drive}:'" | Select-Object Label, @{Label = "SerialNumber"; Expression = {"{0:X}" -f $_.SerialNumber}},Capacity,FreeSpace,FileSystem,BlockSize > ('[Target]\' + $CollCode + '\Metadata\' + $Timestamp + '\Volume information.txt')
robocopy ${drive}:\ "[Target]\$CollCode\$Timestamp"  /MIR /DCOPY:T /a+:R /LOG+:"[Target]\$CollCode\Metadata\$Timestamp\File copy log.txt" /tee /R:3 /W:5 /NP
Pause
```
