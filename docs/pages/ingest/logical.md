---
title: Logical copy
layout: default
nav_order: 4
parent: Ingest
---
We perform logical ingest using the 'robocopy' Windows utility, run via PowerShell. As standard, we use the following settings:

* /MIR                          Deletes files/folders in the destination not present in the source; copies subfolders, including empty ones
* /DCOPY:T                      Copies directory timestamps
* /LOG+:[location] /NP /tee     Outputs logs of copying, appending to any existing log file; suppresses incremental progress display, which otherwise makes logs difficult to read; and outputs to the console window as well as the log file
* /R:3 /W:5                     Restricts retrys to a maximum of 3, with a wait of 5 seconds between each retry. Important as the default values are 1 million retries, with 30 second gaps between each. 
* /a+:R                         Sets all destination files to read-only

```
robocopy [source] "[target]\Media in\Content"  /MIR /DCOPY:T /a+:R /LOG+:"[target]\Media in\Metadata\File copy log.txt" /tee /R:3 /W:5 /NP
```
Where [source] and [target] are appropriate file paths, such as 'E:\' and 'C:\' respectively to copy the contents of the E: drive to a subfolder of '\Media in' on the C: drive. 

## Script Version

We want non-specialist staff to be able to perform our standard ingest procedures, but do not want to require them to know specific syntax, or similar. As such, we have put together a short script that walks a user through choosing a source (assuming copying a full drive, such as a CD or memory stick), designating a collection code, after which everything else is automated, putting the output in a standard location, with the log files in a folder alongside. Folders (both content and metadata) are named with minute level timestamps to avoid accidentally writing over previously copied carriers.

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
Where [Target] is replaced by a file path, such as 'C:\'. This is saved as a .PS1 file, and run by a simple right click 'Run with PowerShell'. 

## Resources

Adam Bertram's [Robocopy Commands: Data Migration, Folder Sync, and More](https://adamtheautomator.com/robocopy) resource, alongside the [robocopy documentation](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/robocopy) was very useful in settling on the exact settings we use for this process.