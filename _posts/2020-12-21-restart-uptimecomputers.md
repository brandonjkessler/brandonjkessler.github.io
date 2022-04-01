---
title: Script - Restart-UptimeComputers
Author: Brandon J. Kessler
tags: Software DevOps SysAdmin PowerShell Scripting
category: Script
published: true
---

<h1>{{ page.title }}</h1>

## TL:DR

The script checks to see if a computer’s uptime is greater-than-or-equal-to a variable of days you specify. If it is, then it schedules a time (in 24-hour time) to restart the next day.

<!--more-->
## The Script

My friend and co-worker Rachel Catches-Ford and I came up with this little solution in PowerShell, with some searching on Stack Exchange for how to schedule a restart on the next day.

Link to Github: [https://github.com/RachelCatchesFord/Restart-UptimeComputers](https://github.com/RachelCatchesFord/Restart-UptimeComputers)

```
Param(
    [parameter(Mandatory)][int]$Days,
    [parameter(Mandatory)][int]$Time
)

$OS = Get-wmiobject Win32_OperatingSystem
$Uptime = (Get-Date) - $OS.ConvertToDateTime($OS.LastBootUpTime)
[int]$DaysUp = $Uptime.TotalDays

if($DaysUp -ge $Days){
    $SupportGroup = 'OIT DESKSIDE SUPPORT
'
    $Tomorrow = (Get-Date).AddDays(1).Date.AddHours($Time)
    msg.exe * "***$($SupportGroup)*** Your computer has been up for $($DaysUp) Days. Scheduling a restart for $($Tomorrow)."
    shutdown -r -t ([decimal]::round(($Tomorrow - (Get-Date)).TotalSeconds))
}
```

As you can see, it accepts two parameters, `Days` and `Time`. `Days` is how many days since the last reboot, or uptime. `Time` is the time you want it to restart the next day.

We use WMI to pull the uptime information from the last boot up time. Then we assign that to a variable that is explicitly marked as an Integer so we can get a nice round number.

Then, we compare the uptime to the parameter for Days and if it’s greater than or equal to it, it’ll schedule a restart for the next day. It also sends out a message letting users know what’s going to happen.