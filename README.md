![image](https://github.com/user-attachments/assets/95613e2d-4d57-4722-a384-28f5dcb48110)
#Retrieve APPX update files via WinGet prior to running the APPX Install Script
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
![image](https://github.com/user-attachments/assets/9dd45fca-6cfc-4cc1-8f76-7a5102a2968d)




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
