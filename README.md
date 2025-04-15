#Retrieve APPX update files via WinGet prior to running the APPX Install Script
### AppX IDs for Windows Applications

| **AppX** | **ID** |
|----------|--------|
| Windows NotePad | 9MSMLRH6LZF3 |
| HEIF | 9PMMSR1CGPWG |
| Microsoft Sticky Notes | 9NBLGGH4QGHW |
| Paint | 9PCFS5B6T72H |
| Power Automate Desktop | 9NFTCH6J7FHV |
| Raw Image Extension | 9NCTDW2W1BH8 |
| ToDo | 9NBLGGH5R558 |
| VP9 Video Extensions | 9N4D0MSMP0PT |
| WebMediaExtensions | 9N5TDP8VCMHS |
| WebpImageExtension | 9PG2DK419DRG |
| Photos | 9WZDNCRFJBH4 |
| Clock | 9WZDNCRFJ3PR |
| Calculator | 9WZDNCRFHVN5 |
| Terminal | Microsoft.WindowsTerminal |
| VCLibs UWPDesktop | Microsoft.VCLibs.Desktop.14 |
| HEVC | 9NMZLZ57R3T7 |
| Snipping Tool (Screensketch) | 9MZ95KL8MR0L |
| Microsoft.WindowsAppRuntime.1.3 | Appruntime |
| Microsoft.WindowsAppRuntime.1.5 | Appruntime |
| Onenote | XPFFZHVGQWWLHB |


```
#PowerShell to download all Inbox Apps, AppX
# List of WinGet IDs for Microsoft Inbox apps

$apps = @(
    "9WZDNCRFHVN5",  # Calculator
    "9WZDNCRFHVQM",  # Calendar and Mail
    "9WZDNCRFJBBG",  # Camera
    "9WZDNCRFJ3PR",  # Clock
    "9NFFX4SZZ23L",  # Cortana
    "XPFFTQ037JWMHS",  # Microsoft Edge
    "9WZDNCRFJBMP",  # Microsoft Teams
    "9NBLGGH5R558",  # Microsoft To Do
    "9MSMLRH6LZF3",  # Notepad
    "9PCFS5B6T72H",  # Paint
    "9WZDNCRFJBH4",  # Photos
    "9MZ95KL8MR0L",  # Snipping Tool
    "9NBLGGH4QGHW",  # Sticky Notes
    "9WZDNCRFJ3Q2",  # Weather
    "9WZDNCRFJBMP",  # Groove Music
    "9WZDNCRFJ3Q2",  # Maps
    "9WZDNCRFJ3Q2",  # Movies & TV
    "9WZDNCRFJ3Q2",  # People
    "9WZDNCRFJ3Q2",  # Skype
    "9WZDNCRFJ3Q2"   # Voice Recorder
)

# Install each app
foreach ($app in $apps) {
    winget download--id $app --accept-package-agreements --architecture x64
}
```

# Win11-24H2-AppxInstall
Updating AppX apps for Win11 24H2 set each of these as individual actions within a Task Sequence in MEM

#If Scheduled Tasks are showing history disabled - this fixes that
wevtutil.exe set-log Microsoft-Windows-TaskScheduler/Operational /enabled:true

#Stage Appx Files locally on the machine
#Select Package and set custom path
C:\temp\stage
#save path as a variable
APPX

#Run Command line - Fix Location & Folder Cleanup | Appx
cmd.exe /c powershell.exe -executionpolicy bypass Copy-Item -path %APPX01%\* -destination C:\temp\stage -recurse -force && powershell.exe -executionpolicy bypass Remove-Item %APPX01% -Recurse -Force

#Run PowerShell Script - Add Scheduled Task
#Set PowerShell Execution Policy as BYPASS
$account = "NT AUTHORITY\SYSTEM"
$StartInDirectory = "C:\temp\stage"
$Action = New-ScheduledTaskAction -Execute "powershell" -Argument " -noprofile -nologo -noninteractive -executionpolicy bypass -f C:\temp\stage\APPX\Appx-Install24H2W11v3.ps1" $StartInDirectory
$Trigger = New-ScheduledTaskTrigger -AtLogOn
$Setting = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries

Register-ScheduledTask - TaskName Appx -User $Account -Action $Action -Trigger $Trigger -Settings $Setting -RunLevel Highest

#APPX install script has logic that will parse through the event viewer log to ensure the install script does not run when DefaulUser0 is logged on

#Script logging available C:\Windows\Logs\AppxINSTALL24H2.log
