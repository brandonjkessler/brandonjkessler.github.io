---
title: McAfee to Bitlocker Conversion
Author: Brandon J. Kessler
tags: DevOps SysAdmin Windows
category: Technology
published: true
---

<h1>{{ page.title }}</h1>

Recently at work we were tasked with migrating over 2500 endpoints from McAfee Encryption to Bitlocker. The challenge was that we have multiple sites with varying levels of bandwidth and some users working remotely. We also needed to back up the key not only to AD, but also to a .CSV on a network share that only our desk side team has access to. And we had until the End of March to do it while being informed of the change over in January.
<!--more-->
## Overview

The main part of the solution was a set of scripts that tested for McAfee encryption, and once it was no longer encrypted kick off Bitlocker. We used Compliance Baselines and a Task Sequence to upgrade everything. The final solution you see was after multiple iterations and new complications added in to the mix. Our solution is not pretty or clean, but it works!

We ended up doing the deployment in two phases. Phase 1 was the initial BIOS configuration changes via SCCM and Powershell, and placing the scripts on the device and creating a scheduled task to monitor encryption status using another script. Phase 2 was much simpler, and was us mass decrypting devices using the ePO. Once the device was no longer encrypted with McAfee, our monitoring script from Phase 1 would detect the change and then reboot the machine, and create a scheduled task to enable Bitlocker.

We created multiple device collections to filter devices to the appropriate points of the deployment. I highly recommend creating an “All Laptops and Tablets” collection to use as a limiting collection unless you also need to encrypt desktops.

The basic flow of the conversion was roughly the following:

1.  System is placed in collection to apply Task Sequence (TS).
    1.  The system is placed via Configuration Baselines.
2.  Phase 1 TS is run on device
    1.  Sets UEFI/BIOS to turn on TPM, enable Secure Boot
    2.  If Legacy Enabled, begins MBR to GPT conversion
3.  System is placed in new Device Collection once TS is run successfully
4.  Begin decryption in ePO of devices in Success Device collection
5.  Wait

That’s really it! But, it took a lot of breaking things for us to figure some things out. It also doesn’t help that we have no control over the ePO and it is all handled by our SecOps team. My good friend and coworker Rachel and I spent a lot of iterations testing, breaking, and trying again before we got all the parts to work. It was also trying to hit a moving target. So what you’ll see for our solution absolutely could be improved upon and made sleeker, faster, and probably more robust. I hope it helps you get a head start!

## Device Collections

We used multiple device collections throughout the process. These are the ones that you could get by with. We had more than this but by the end this is what mattered. We did have Pilot – <DEVICE COLLECTION> for most of these that used the limiting group of our normal Pilot group.

-   Bitlocker – Exclusions
    -   Use for Exclusions
-   OSD – Bitlocker Enabled\_All Laptops\_Compliant
    -   Use for Exclusions
-   OSD – Bitlocker Enabled\_All Laptops\_Noncompliant
    -   This is where the TS is applied
    -   Exclude: Bitlocker – Exclusions, OSD – Bitlocker Enabled\_All Laptops\_Compliant, MBR TS – Succeeded
-   MBR TS – Succeeded
    -   Use a Query to pull completed items into this
    -   Exclude: OSD – Bitlocker Enabled\_All Laptops\_Compliant

### Device Collection – Queries

#### All Laptops

```
select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_SYSTEM_ENCLOSURE on SMS_G_System_SYSTEM_ENCLOSURE.ResourceID = SMS_R_System.ResourceId where SMS_G_System_SYSTEM_ENCLOSURE.ChassisTypes in ("8","9","10","11","12","14","18","21","30","31","32")
```

#### MBR TS – Succedded

```
select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ResourceId in (select ResourceID from SMS_ClientAdvertisementStatus where AdvertisementID = "<YOUR TS ID>" and LastStatusMessageID=11171)
```

## Configuration Baselines and Configuration Items

We used the HP Client Management Script Library (HP CMSL) to achieve a large part of our results. We are exclusively an HP shop (with a few random Surface Pro 3’s and 4’s in the mix) so this worked well for us. We created a Configuration Item (CI) to detect if the CMSL was installed. We then followed up with detecting if the Legacy Boot was enabled, Secure Boot Enabled, and TPM Enabled. We could then build a configuration baseline to detect these. We also created CIs for Bitlocker and McAfee detection. Check at the bottom of the blog for the scripts.

We used OSD – Bitlocker Enabled to exclude all devices that were already running Bitlocker.

### Configuration Items

-   Encryption – Bitlocker
-   Encryption – McAfee
-   Hardware – HP BIOS – Legacy Boot enabled
-   Hardware – Secure Boot Enabled
-   Hardware – TPM – Enabled

### Configuration Baselines

-   OSD – Bitlocker Enabled
-   OSD – Legacy Boot Disabled
-   OSD – McAfee Decrypted

## Scripts

All the scripts on the client must be run from the same folder, in the order they appear below. The idea is that the Convert-McAfeeToBitlocker.ps1 will run on startup or logon as a scheduled task. It will check every 5 minutes to see if McAfee Encryption is still active on the device. Once it detects that McAfee is decrypted, it will kick off the Enable-Bitlocker.ps1 script.

### Scripts on Client

Do-AllTheThings.ps1

```
## Deletes the existing ReAgent XML, copies all of the files needed to the Bitlocker Directory.

$OutputDir = "C:\Updates\Bitlocker"

cmd.exe /c GPUpdate /Force

Remove-Item -Path "$env:windir\System32\Recovery\ReAgent.xml" -Recurse -Force -Verbose
Copy-Item -Path "$PSScriptRoot\ReAgent.xml" -Destination "$env:windir\System32\Recovery\ReAgent.xml" -Recurse -Force -Verbose
Remove-Item -Path "$OutputDir" -Recurse -Force -Verbose
New-Item -Path "$OutputDir" -ItemType Directory -Force -Verbose
Copy-Item -Path "$PSScriptRoot\*.*" -Destination "$OutputDir" -Recurse -Force -Verbose
Copy-Item -Path "$OutputDir\Clean-Scripts.ps1" -Destination "C:\Updates"
```

Import-ScheduledTask.ps1

```
Param(
    [Parameter(mandatory=$true)][string]$XML,
    [Parameter(mandatory=$true)][string]$TaskName
)

Register-ScheduledTask -Xml (get-content $XML | out-string) -User USERACCOUNT -Password "PASSWORD" -TaskName $TaskName
```

Convert-McAfeeToBitlocker.ps1

```
##################################################################################################
# File Version: 1.1.0
# Author: Brandon Kessler
# Author: Rachel Catches-Ford
# Description: Tests for McAfee and detects whether or not the Device is encrypted with McAfee
# or bitlocker
##################################################################################################


Start-Transcript -Path 'C:\Windows\Logs\McAfeeToBitlocker.log' -Append

##################################################################################################
#----------------------------------------- Variables --------------------------------------------#
##################################################################################################

$OutputDir = "OUTPUTDIR"

#Set some variables for Time
$Seconds = 300 # This variable is used for looping and also for Start-Wait later in the script.
$Minutes = $Seconds/60 # Makes it human readable in minutes


## Get McAfee Encryption information
function Get-McAfeeEncryption{
    $EncryptionRegMcAfee = (Get-ItemProperty -Path 'HKLM:\SOFTWARE\WOW6432Node\McAfee EndPoint Encryption\MfeEpePC\Status')
    $EncryptionActive = $EncryptionRegMcAfee.Activated
    $EncryptionQueryProgress = $EncryptionRegMcAfee.CryptState
    $EncryptionState = $EncryptionQueryProgress | select-string -Pattern 'Volume=C:,State=(\w+)' | ForEach-Object {$_.Matches.Groups[1].Value}

    Return $EncryptionState
}

function Send-McAfeeStatus{
    cmd.exe /c "`"C:\Program Files\McAfee\Agent\cmdagent.exe`" -p"
    Write-Host 'Send Events'
    cmd.exe /c "`"C:\Program Files\McAfee\Agent\cmdagent.exe`" -f"
    Write-Host 'Check for New Policies'
    cmd.exe /c "`"C:\Program Files\McAfee\Agent\cmdagent.exe`" -c"
    Write-Host 'Enforce Policies Locally'
    cmd.exe /c "`"C:\Program Files\McAfee\Agent\cmdagent.exe`" -e"
}


if(!(Test-Path -Path 'HKLM:\SOFTWARE\Custom\McAfeeToBitlocker')){
    New-Item -Path 'HKLM:\SOFTWARE\Custom\' -Name 'McAfeeToBitlocker' -Force
    New-ItemProperty -Path 'HKLM:\SOFTWARE\Custom\McAfeeToBitlocker' -Name 'McAfee' -Value 'Unknown'
    New-ItemProperty -Path 'HKLM:\SOFTWARE\Custom\McAfeeToBitlocker' -Name 'Bitlocker' -Value 'Unknown'
    New-ItemProperty -Path 'HKLM:\SOFTWARE\Custom\McAfeeToBitlocker' -Name 'TestAttempts' -Value "$Attempt"
}


if((Get-BitLockerVolume -MountPoint 'C:').VolumeStatus -ne 'FullyDecrypted'){
    Set-ItemProperty -Path 'HKLM:\SOFTWARE\Custom\McAfeeToBitlocker' -Name 'McAfee' -Value 'Decrypted'
    Set-ItemProperty -Path 'HKLM:\SOFTWARE\Custom\McAfeeToBitlocker' -Name 'Bitlocker' -Value 'Encrypted'
    & "$PSScriptRoot\Enable-Bitlocker.ps1"
}elseif(Test-Path -Path 'HKLM:\SOFTWARE\WOW6432Node\McAfee EndPoint Encryption\MfeEpePC\Status'){
    Write-Host 'Testing McAfee Encryption Status'
    $Attempt = 0 ## Just to see on average how many attempts there were
    ## While loop will test for encryption status of McAfee before proceeding with the script.

    While((Get-McAfeeEncryption) -ne 'Decrypted'){
        Set-ItemProperty -Path 'HKLM:\SOFTWARE\Custom\McAfeeToBitlocker' -Name 'McAfee' -Value 'Encrypted'
        $Attempt += 1
        Write-Host "Current time is $(Get-Date -Format HH:mm:ss)"
        Write-Output "Now proceeding with Attempt $Attempt"
        Set-ItemProperty -Path 'HKLM:\SOFTWARE\Custom\McAfeeToBitlocker' -Name 'TestAttempts' -Value "$Attempt"
        Write-Warning 'System is still encrypted with McAfee.'
        Write-Host 'Collect and Send Properties'
        Send-McAfeeStatus
        Write-Host "System will Wait $Minutes Minutes before trying again at $((Get-Date).AddMinutes(5).ToString('HH:mm:ss'))."
        Start-sleep -Seconds $Seconds
    } # End While

    if((Get-ItemProperty -Path 'HKLM:\SOFTWARE\Custom\McAfeeToBitlocker').Bitlocker -eq 'Waiting'){
        Set-ItemProperty -Path 'HKLM:\SOFTWARE\Custom\McAfeeToBitlocker' -Name 'Bitlocker' -Value 'Encrypting'
        & "$PSScriptRoot\Enable-Bitlocker.ps1"
        Exit 0
    }elseif(((Get-McAfeeEncryption) -eq 'Decrypted') -and ((Get-BitLockerVolume -MountPoint 'C:').VolumeStatus -eq 'FullyDecrypted')){
        Set-ItemProperty -Path 'HKLM:\SOFTWARE\Custom\McAfeeToBitlocker' -Name 'McAfee' -Value 'Decrypted'
        Set-ItemProperty -Path 'HKLM:\SOFTWARE\Custom\McAfeeToBitlocker' -Name 'Bitlocker' -Value 'Waiting'
        ## Alert user that a restart will happen
        cmd /c "msg * /TIME:120 Your computer will restart in $Minutes minutes to begin encrypting with Bitlocker."
        cmd /c "shutdown /r /t $Seconds"
    } ## End If
} elseif(((Get-BitLockerVolume -MountPoint 'C:').VolumeStatus -eq 'FullyDecrypted') -and ((Test-Path -Path 'HKLM:\SOFTWARE\WOW6432Node\McAfee EndPoint Encryption\MfeEpePC\Status') -eq $false)) {
    Write-Warning 'Encryption not found on device.'
    Set-ItemProperty -Path 'HKLM:\SOFTWARE\Custom\McAfeeToBitlocker' -Name 'McAfee' -Value 'Decrypted'
    Set-ItemProperty -Path 'HKLM:\SOFTWARE\Custom\McAfeeToBitlocker' -Name 'Bitlocker' -Value 'Encrypting'
    & "$PSScriptRoot\Enable-Bitlocker.ps1"
    Exit 0
} else {
    Write-Error 'No Encryption found.'
    Write-Error 'Could not encrypt device or other fatal error.'
    Set-ItemProperty -Path 'HKLM:\SOFTWARE\Custom\McAfeeToBitlocker' -Name 'McAfee' -Value 'Failed'
    Set-ItemProperty -Path 'HKLM:\SOFTWARE\Custom\McAfeeToBitlocker' -Name 'Bitlocker' -Value 'Failed'
    & "$PSScriptRoot\Clean-Scripts.ps1"

    Exit 1
}
Stop-Transcript
```

Enable-Bitlocker.ps1

```
##################################################################################################
# File Version: 1.1.0
# Author: Brandon Kessler
# Author: Rachel Catches-Ford
# Description: Tests for McAfee and detects whether or not the Device is encrypted with McAfee
# or bitlocker
##################################################################################################


Start-Transcript -Path 'C:\Windows\Logs\McAfeeToBitlocker.log' -Append

##################################################################################################
#----------------------------------------- Variables --------------------------------------------#
##################################################################################################

$OutputDir = "OUTPUTDIR"

#Set some variables for Time
$Seconds = 300 # This variable is used for looping and also for Start-Wait later in the script.
$Minutes = $Seconds/60 # Makes it human readable in minutes

function Start-Bitlocker{
    Write-Host 'Encrypting with Bitlocker.'
    #cmd.exe /c "msg.exe * /w Your computer is Encrypting with BitLocker. Please do not Restart or Shutdown your computer. If you do, you will need to contact the Service Desk to get into your computer."
    enable-bitlocker -mountpoint "C:" -EncryptionMethod XtsAes256 -skiphardwaretest -recoverypasswordprotector -Verbose
    ## Check for TPM Key Protector and Add if not there
    $KeyProtectorArray = @()
    (Get-BitLockerVolume -MountPoint 'C:').KeyProtector | Select-Object KeyProtectorType | ForEach-Object {$KeyProtectorArray += $_.KeyProtectorType}
    if(($KeyProtectorArray -notcontains 'Tpm')){ ## Verify that TPM was also set
        Add-BitlockerKeyProtector -MountPoint 'C:' -TpmProtector -Verbose
    }
}

function Backup-BitlockerInfo{
    Param(
        [string]$Path = $PSScriptRoot
    )
    
    $blv = Get-BitlockerVolume -MountPoint "C:"
    $RecoveryGUID = $blv.keyprotector | Where-Object{$_.keyprotectortype -eq 'recoverypassword'} | Select-Object -expandproperty keyprotectorid
    $RecoveryPassword = $blv.keyprotector | Where-Object{$_.keyprotectortype -eq 'recoverypassword'} | Select-Object -expandproperty RecoveryPassword

    # Create a computer object to hold information
    $ComputerInfo = [PSCustomObject]@{
        "Computer Name" = $env:computername
        "Serial Number" = Get-WmiObject -class Win32_BIOS | Select-Object -ExpandProperty SerialNumber
        "Recovery GUID" = $RecoveryGUID
        "Recovery Key" = $RecoveryPassword
        "Date" = get-date -Format yyyy-MM-dd
    } # End ComputerInfo Object

    ## Backup to AD
    Backup-BitLockerKeyProtector -MountPoint "C:" -KeyProtectorId $RecoveryGUID -Verbose

    ## Create a CSV file
    $ComputerInfo | Export-Csv -Path "$Path\Bitlocker.csv" -Append -NoTypeInformation


}

if(!(Test-Path -Path 'HKLM:\SOFTWARE\Custom\McAfeeToBitlocker')){
    New-Item -Path 'HKLM:\SOFTWARE\Custom\' -Name 'McAfeeToBitlocker' -Force
    New-ItemProperty -Path 'HKLM:\SOFTWARE\Custom\McAfeeToBitlocker' -Name 'McAfee' -Value 'Unknown'
    New-ItemProperty -Path 'HKLM:\SOFTWARE\Custom\McAfeeToBitlocker' -Name 'Bitlocker' -Value 'Unknown'
}

while((Test-Path -Path $OutputDir) -ne $true){ # Test for Connection to the domain

    ## Alert them that they need to connect to the network
    cmd /c "msg.exe * /TIME:120 Your computer is not attached to the network. Please dock your computer, or connect to the Wireless Network. If you are at a remote location, please connect to the VPN."
    Write-Warning 'System is not connected to the network.'
    Write-Warning "System will attempt again in $Minutes minutes."
    Write-Host "Starting time is $(Get-Date -Format HH:mm)"
    Start-Sleep -Seconds $Seconds
}

if((Get-BitLockerVolume -MountPoint 'C:').VolumeStatus -ne 'FullyDecrypted'){
    Set-ItemProperty -Path 'HKLM:\SOFTWARE\OIT-CDHS\McAfeeToBitlocker' -Name 'Bitlocker' -Value 'Encrypted'
    Write-Host 'System already encrypting/encrypted with Bitlocker. Backing up Keys.'
    Backup-BitlockerInfo -Path $OutputDir
} else {
    Set-ItemProperty -Path 'HKLM:\SOFTWARE\OCustom\McAfeeToBitlocker' -Name 'Bitlocker' -Value 'Encrypted'
    Start-Bitlocker
    ##Backup Recovery Key GUID to Active Directory
    ## Write info to a csv file on a shared drive
    Write-Host 'Backing up Bitlocker Key to AD'
    Write-Host "Writing information for Computer $ENV:computername to $OutputDir\Bitlocker.csv"
    Backup-BitlockerInfo -Path $OutputDir  
}

## Run Cleanup Scripts Script...Script
& "C:\Updates\Clean-Scripts.ps1"

Stop-Transcript
```

Clean-Scripts.ps1

```
## Cleanup scripts

cmd.exe /c "C:\Updates\Bitlocker\Remove-MDE.bat"

## Cleanup McAfee to Bitlocker
$TaskName = 'Convert-McAfeeToBitlocker'
if(Get-ScheduledTask -TaskName $TaskName){
    Write-Warning "Unregistering the scheduled task $TaskName."
    Unregister-ScheduledTask -TaskName $TaskName -Confirm:$false
}

## Cleanup McAfee to Bitlocker
$TaskName = 'Get-LegacyBootStatus'
if(Get-ScheduledTask -TaskName $TaskName){
    Write-Warning "Unregistering the scheduled task $TaskName."
    Unregister-ScheduledTask -TaskName $TaskName -Confirm:$false
}

set-location -Path "C:\Updates"

## Delete Bitlocker folder after running conversion
```

### Configuration Items Settings

#### Encryption – Bitlocker

**Discovery Script**

```
## Get bitlocker info
$blv = Get-BitLockerVolume -MountPoint 'C:'

## Check for Volume Status
if($blv.VolumeStatus -eq 'FullyEncrypted'){
    $Encrypted = $true
}

## Check for KeyProtectors
$KeyProtectorArray = @()
$blv.KeyProtector | Select-Object KeyProtectorType | ForEach-Object {$KeyProtectorArray += $_.KeyProtectorType}
if(($KeyProtectorArray -contains 'Tpm') -and ($KeyProtectorArray -contains 'RecoveryPassword')){ ## Verify that TPM and RecoveryPassword are set
    $KeyCorrect = $true
}

if(($Encrypted -eq $true) -and ($KeyCorrect -eq $true)){
    Write-Output 'Encrypted'
} else {
    Write-Output 'Not Encrypted'
}
```

**Compliance Rule**

“The value returned by the specified script:” Equals “Encrypted”

#### Encryption – McAfee

**Discovery Script**

```
## Test for McAfee Encryption
If(Test-Path -Path 'HKLM:\SOFTWARE\WOW6432Node\McAfee EndPoint Encryption\MfeEpePC\Status'){
     $EncryptionReg = (Get-ItemProperty -Path 'HKLM:\SOFTWARE\WOW6432Node\McAfee EndPoint Encryption\MfeEpePC\Status')

     $EncryptionActive = $EncryptionReg.Activated

     $EncryptionQueryProgress = $EncryptionReg.CryptState

     ## Queries the Crypt State.
     $EncryptionState = $EncryptionQueryProgress | select-string -Pattern 'Volume=C:,State=(\w+)' | ForEach-Object {$_.Matches.Groups[1].Value}

     ## Check for McAfee Encryption
     if($EncryptionState -ne 'Decrypted'){ ## Start if 2
          Write-Output $true
     } Else {
         Write-Output $false
     }## End if 2
} Else { ## End if 1
Write-Output $False
}
```

**Compliance Rules**

“The value returned by the specified script:” Equals “False”

#### Hardware – HP BIOS – Legacy Boot enabled

**Discovery Script**

```
## BEGIN Taken from: https://www.configjon.com/hp-bios-settings-management/

#Connect to the HP_BIOSEnumeration WMI class
$SettingList = Get-WmiObject -Namespace root\HP\InstrumentedBIOS -Class HP_BIOSEnumeration

#Connect to the HP_BIOSSetting WMI class
$HPBiosSettings = Get-WmiObject -Namespace root\HP\InstrumentedBIOS -Class HP_BIOSSetting

#Connect to the HP_BIOSSettingInterface WMI class
$Interface = Get-WmiObject -Namespace root\HP\InstrumentedBIOS -Class HP_BIOSSettingInterface

$LegacyValue = $HPBiosSettings | Where-Object{$_.Name -like "*Legacy Support*"} | Select -Property "CurrentValue"

if($LegacyValue -like "*Legacy Support Enable*"){
    Write-Output "Enabled"
} else {
    Write-Output "Disabled"
}
```

**Compliance Rules**

“The value returned by the specified script:” Equals “Disabled”

#### Hardware – Secure Boot Enabled

**Discovery Script**

```
## Test for Secure Boot

If((Confirm-SecureBootUEFI) -eq $true){
    Write-Output 'Enabled'
} Else {
    Write-Output 'Disabled'
}
```

**Remediation Script**

```
## BEGIN Taken from: https://www.configjon.com/hp-bios-settings-management/

#Connect to the HP_BIOSEnumeration WMI class
$SettingList = Get-WmiObject -Namespace root\HP\InstrumentedBIOS -Class HP_BIOSEnumeration

#Connect to the HP_BIOSSetting WMI class
$HPBiosSettings = Get-WmiObject -Namespace root\HP\InstrumentedBIOS -Class HP_BIOSSetting

#Connect to the HP_BIOSSettingInterface WMI class
$Interface = Get-WmiObject -Namespace root\HP\InstrumentedBIOS -Class HP_BIOSSettingInterface

## END Taken from: https://www.configjon.com/hp-bios-settings-management/


## set Variables

$Settings = (

    'Configure Legacy Support and Secure Boot;Legacy Support Disable and Secure Boot Enable' #Legacy Support Enable and Secure Boot Disable,Legacy Support Disable and Secure Boot Enable,Legacy Support Disable and Secure Boot Disable

)

## Set a specific HP BIOS setting
## https://github.com/ConfigJon/Firmware-Management/blob/master/HP/Manage-HPBiosSettings.ps1
Function Set-HPBiosSetting
{
    param (
        [Parameter(Mandatory=$true)][ValidateNotNullOrEmpty()][String]$Name,
        [Parameter(Mandatory=$true)][ValidateNotNullOrEmpty()][String]$Value,
        [Parameter(Mandatory=$false)][ValidateNotNullOrEmpty()][String]$Password
    )

    #Ensure the specified setting exists and get the possible values
    $CurrentSetting = $SettingList | Where-Object Name -eq $Name | Select-Object -ExpandProperty Value
    if ($NULL -ne $CurrentSetting)
    {
        #Split the current values
        $CurrentSettingSplit = $CurrentSetting.Split(',')

        #Find the currently set value
        $Count = 0
        while($Count -lt $CurrentSettingSplit.Count)
        {
            if ($CurrentSettingSplit[$Count].StartsWith('*'))
            {
                $CurrentValue = $CurrentSettingSplit[$Count]
                break
            }
            else
            {
                $Count++
            }
        }
        #Setting is already set to specified value
        if ($CurrentValue.Substring(1) -eq $Value)
        {
            Write-Warning "Setting $Name is already set to $Value"
            #$Script:AlreadySet++
        }
        #Setting is not set to specified value
        else
        {
            if (!([String]::IsNullOrEmpty($Password)))
            {
                $SettingResult = ($Interface.SetBIOSSetting($Name,$Value,"<utf-16/>" + $Password)).Return
            }
            else
            {
                $SettingResult = ($Interface.SetBIOSSetting($Name,$Value)).Return
            }
            

            if ($SettingResult -eq 0)
            {
                Write-Host "Successfully set $Name to $Value"
                #$Script:SuccessSet++
            }
            else
            {
                Write-Error "Failed to set $Name to $Value"
                #$Script:FailSet++
            }
        }
    }
    #Setting not found
    else
    {
        Write-Warning "Setting $Name not found" 
        #$Script:NotFound++
    }
}

#Set HP BIOS settings - password is not set

ForEach($Setting in $Settings){
    $Data = $Setting.Split(';')
    Set-HPBiosSetting -Name $Data[0].Trim() -Value $Data[1].Trim()
}
```

**Compliance Rules**

“The value returned by the specified script:” Equals “Enabled”

#### Hardware – TPM – Enabled

**Discovery Script**

TPM – Present

```
## Test for TPM
if((Get-Tpm).TpmPresent -eq $true){
    Write-Output 'Enabled'
} else {
    Write-Output 'Disabled'
}
```

TPM – Ready

```
## Test for TPM
if((Get-TPM).TpmReady -eq $true){
    Write-Output 'Enabled'
} else {
    Write-Output 'Disabled'
}
```

**Remediation Script**

TPM – Present

```
## BEGIN Taken from: https://www.configjon.com/hp-bios-settings-management/

#Connect to the HP_BIOSEnumeration WMI class
$SettingList = Get-WmiObject -Namespace root\HP\InstrumentedBIOS -Class HP_BIOSEnumeration

#Connect to the HP_BIOSSetting WMI class
$HPBiosSettings = Get-WmiObject -Namespace root\HP\InstrumentedBIOS -Class HP_BIOSSetting

#Connect to the HP_BIOSSettingInterface WMI class
$Interface = Get-WmiObject -Namespace root\HP\InstrumentedBIOS -Class HP_BIOSSettingInterface

## END Taken from: https://www.configjon.com/hp-bios-settings-management/


## set Variables

$Settings = (

    ## Enable Secure Boot
    "SecureBoot Security Level;Change", #Change,View,Hide
    'Configure Legacy Support and Secure Boot;Legacy Support Disable and Secure Boot Enable', #Legacy Support Enable and Secure Boot Disable,Legacy Support Disable and Secure Boot Enable,Legacy Support Disable and Secure Boot Disable
    "SecureBoot;Enable", #Enable,Disable

    ### Enable TPM
    ## Security Levels
    "TPM Device Security Level;Change" , #Change, View, Hide
    "Activate TPM On Next Boot Security Level;Change", #Change, View, Hide
    "TPM Activation Policy Security Level;Change" ,  #Change, View, Hide
    "OS Management of TPM Security Level;Change" ,  #Change, View, Hide
    "Reset of TPM from OS Security Level;Change" ,  #Change, View, Hide
    "TPM Embedded Security Security Level;Change" ,  #Change, View, Hide
    "Embedded Security Device;Device available", #Device hidden,Device available
    "Embedded Security Device Availability;Available", #Available,Hidden
    "OS management of Embedded Security Device;Enable", #Enable,Disable
    "OS Management of TPM;Enable", #Disable,Enable
    "Reset of Embedded Security Device through OS;Enable", #Disable,Enable
    "Reset of TPM from OS;Enable", #Disable,Enable
    "TPM Device;Available", #Hidden,Available
    "TPM State;Enable", #Disable,Enable
    "Activate TPM On Next Boot;Enable" , #Disable, Enable
    "TPM Activation Policy;No prompts" , #No prompts, F1 to boot, Allow user to reject prompts
    ## Clear TPM
    "Clear TPM;On next boot", #No,On next boot
    "TPM Clear;Clear", #Do not Clear,Clear
    "TPM Reset to Factory Defaults;Yes" , #Yes, No
    ## Physical Presence Interface Settings
    "Physical Presence Interface;Enable" #Disable,Enable
    "Tpm No PPI maintenance;Enable", #Disable,Enable
    "Tpm No PPI provisioning;Enable", #Disable,Enable
    "Tpm PPI policy changed by OS allowed;Enable" #Disable,Enable
    
)

## Set a specific HP BIOS setting
## https://github.com/ConfigJon/Firmware-Management/blob/master/HP/Manage-HPBiosSettings.ps1
Function Set-HPBiosSetting
{
    param (
        [Parameter(Mandatory=$true)][ValidateNotNullOrEmpty()][String]$Name,
        [Parameter(Mandatory=$true)][ValidateNotNullOrEmpty()][String]$Value,
        [Parameter(Mandatory=$false)][ValidateNotNullOrEmpty()][String]$Password
    )

    #Ensure the specified setting exists and get the possible values
    $CurrentSetting = $SettingList | Where-Object Name -eq $Name | Select-Object -ExpandProperty Value
    if ($NULL -ne $CurrentSetting)
    {
        #Split the current values
        $CurrentSettingSplit = $CurrentSetting.Split(',')

        #Find the currently set value
        $Count = 0
        while($Count -lt $CurrentSettingSplit.Count)
        {
            if ($CurrentSettingSplit[$Count].StartsWith('*'))
            {
                $CurrentValue = $CurrentSettingSplit[$Count]
                break
            }
            else
            {
                $Count++
            }
        }
        #Setting is already set to specified value
        if ($CurrentValue.Substring(1) -eq $Value)
        {
            Write-Warning "Setting $Name is already set to $Value"
            #$Script:AlreadySet++
        }
        #Setting is not set to specified value
        else
        {
            if (!([String]::IsNullOrEmpty($Password)))
            {
                $SettingResult = ($Interface.SetBIOSSetting($Name,$Value,"<utf-16/>" + $Password)).Return
            }
            else
            {
                $SettingResult = ($Interface.SetBIOSSetting($Name,$Value)).Return
            }
            

            if ($SettingResult -eq 0)
            {
                Write-Host "Successfully set $Name to $Value"
                #$Script:SuccessSet++
            }
            else
            {
                Write-Error "Failed to set $Name to $Value"
                #$Script:FailSet++
            }
        }
    }
    #Setting not found
    else
    {
        Write-Warning "Setting $Name not found" 
        #$Script:NotFound++
    }
}

#Set HP BIOS settings - password is not set

ForEach($Setting in $Settings){
    $Data = $Setting.Split(';')
    Set-HPBiosSetting -Name $Data[0].Trim() -Value $Data[1].Trim()
}

```

TPM – Ready

```
Initialize-TPM -AllowClear
```

**Compliance Rules**

TPM – Present

“The value returned by the specified script:” Equals “Enabled”

TPM – Ready

“The value returned by the specified script:” Equals “Enabled”

## Task Sequence

Below is the Task sequence we finally ended up with after multiple iterations.

![](/assets/screenshots/Microsoft.ConfigurationManagement_DyTF3EAKCb.png)

Copy Files and Replace ReAgent XML

![](/assets/screenshots/Microsoft.ConfigurationManagement_8ikLEA9Aor-1.png)

Import Scheduled Task

![](/assets/screenshots/Microsoft.ConfigurationManagement_fZgR2gnKbh.png)

### Scripts

Check for McAfee Decrypted on MBR

```
function Get-McAfeeEncryption{
    $EncryptionRegMcAfee = (Get-ItemProperty -Path 'HKLM:\SOFTWARE\WOW6432Node\McAfee EndPoint Encryption\MfeEpePC\Status')
    $EncryptionActive = $EncryptionRegMcAfee.Activated
    $EncryptionQueryProgress = $EncryptionRegMcAfee.CryptState
    $EncryptionState = $EncryptionQueryProgress | select-string -Pattern 'Volume=C:,State=(\w+)' | ForEach-Object {$_.Matches.Groups[1].Value}

    Return $EncryptionState
}

$Disk0PartStyle = (get-disk -Number 0).PartitionStyle

If(((Get-McAfeeEncryption) -ne "Decrypted") -and ($Disk0PartStyle -eq "MBR")){
    Write-Output "McAfee Encrypted on MBR"
    Exit 1
}else{
    Write-Output "Decrypted and Ready"
    Exit 0
}
```

Backup Bitlocker Key if Bitlocker is Enabled

```
if((Get-BitLockerVolume -MountPoint 'C:').VolumeStatus -ne 'FullyDecrypted'){
$blv = Get-BitlockerVolume -MountPoint "C:"
    $RecoveryGUID = $blv.keyprotector | Where-Object{$_.keyprotectortype -eq 'recoverypassword'} | Select-Object -expandproperty keyprotectorid
    $RecoveryPassword = $blv.keyprotector | Where-Object{$_.keyprotectortype -eq 'recoverypassword'} | Select-Object -expandproperty RecoveryPassword

Backup-BitLockerKeyProtector -MountPoint "C:" -KeyProtectorId $RecoveryGUID


Write-Output "Bitlocker is enabled, backing up key to AD."

Exit 1
}else{
Write-Output "Bitlocker is not enabled, Continue with TS."
Exit 0
}
```

Detect-MBR.ps1

```
$Disk0PartStyle = (get-disk -Number 0).PartitionStyle

If($Disk0PartStyle -eq "MBR"){
    cmd.exe /c "MBR2GPT.exe /convert /disk:0 /allowFullOS /logs:C:\Windows\Logs"
    Exit 0
}Elseif($Disk0PartStyle -eq "GPT"){
    Exit 0
}else{
    Exit 1
}
```

Enable Secure-Boot and TPM

```
## BEGIN Taken from: https://www.configjon.com/hp-bios-settings-management/

#Connect to the HP_BIOSEnumeration WMI class
$SettingList = Get-WmiObject -Namespace root\HP\InstrumentedBIOS -Class HP_BIOSEnumeration

#Connect to the HP_BIOSSetting WMI class
$HPBiosSettings = Get-WmiObject -Namespace root\HP\InstrumentedBIOS -Class HP_BIOSSetting

#Connect to the HP_BIOSSettingInterface WMI class
$Interface = Get-WmiObject -Namespace root\HP\InstrumentedBIOS -Class HP_BIOSSettingInterface

## END Taken from: https://www.configjon.com/hp-bios-settings-management/


## set Variables

$Settings = (
    ## Enable Secure Boot
    "SecureBoot Security Level;Change", #Change,View,Hide
    'Configure Legacy Support and Secure Boot;Legacy Support Disable and Secure Boot Enable', #Legacy Support Enable and Secure Boot Disable,Legacy Support Disable and Secure Boot Enable,Legacy Support Disable and Secure Boot Disable
    "SecureBoot;Enable", #Enable,Disable

    ### Enable TPM
    ## Security Levels
    "TPM Device Security Level;Change" , #Change, View, Hide
    "Activate TPM On Next Boot Security Level;Change", #Change, View, Hide
    "TPM Activation Policy Security Level;Change" ,  #Change, View, Hide
    "OS Management of TPM Security Level;Change" ,  #Change, View, Hide
    "Reset of TPM from OS Security Level;Change" ,  #Change, View, Hide
    "TPM Embedded Security Security Level;Change" ,  #Change, View, Hide
    "Embedded Security Device;Device available", #Device hidden,Device available
    "Embedded Security Device Availability;Available", #Available,Hidden
    "OS management of Embedded Security Device;Enable", #Enable,Disable
    "OS Management of TPM;Enable", #Disable,Enable
    "Reset of Embedded Security Device through OS;Enable", #Disable,Enable
    "Reset of TPM from OS;Enable", #Disable,Enable
    "TPM Device;Available", #Hidden,Available
    "TPM State;Enable", #Disable,Enable
    "Activate TPM On Next Boot;Enable" , #Disable, Enable
    "TPM Activation Policy;No prompts" , #No prompts, F1 to boot, Allow user to reject prompts
    ## Clear TPM
    "Clear TPM;On next boot", #No,On next boot
    "TPM Clear;Clear", #Do not Clear,Clear
    "TPM Reset to Factory Defaults;Yes" , #Yes, No
    ## Physical Presence Interface Settings
    "Physical Presence Interface;Enable" , #Disable,Enable
    "Tpm No PPI maintenance;Enable", #Disable,Enable
    "Tpm No PPI provisioning;Enable", #Disable,Enable
    "Tpm PPI policy changed by OS allowed;Enable" #Disable,Enable
    
)

## Set a specific HP BIOS setting
## https://github.com/ConfigJon/Firmware-Management/blob/master/HP/Manage-HPBiosSettings.ps1
Function Set-HPBiosSetting
{
    param (
        [Parameter(Mandatory=$true)][ValidateNotNullOrEmpty()][String]$Name,
        [Parameter(Mandatory=$true)][ValidateNotNullOrEmpty()][String]$Value,
        [Parameter(Mandatory=$false)][ValidateNotNullOrEmpty()][String]$Password
    )

    #Ensure the specified setting exists and get the possible values
    $CurrentSetting = $SettingList | Where-Object Name -eq $Name | Select-Object -ExpandProperty Value
    if ($NULL -ne $CurrentSetting)
    {
        #Split the current values
        $CurrentSettingSplit = $CurrentSetting.Split(',')

        #Find the currently set value
        $Count = 0
        while($Count -lt $CurrentSettingSplit.Count)
        {
            if ($CurrentSettingSplit[$Count].StartsWith('*'))
            {
                $CurrentValue = $CurrentSettingSplit[$Count]
                break
            }
            else
            {
                $Count++
            }
        }
        #Setting is already set to specified value
        if ($CurrentValue.Substring(1) -eq $Value)
        {
            Write-Warning "Setting $Name is already set to $Value"
            #$Script:AlreadySet++
        }
        #Setting is not set to specified value
        else
        {
            if (!([String]::IsNullOrEmpty($Password)))
            {
                $SettingResult = ($Interface.SetBIOSSetting($Name,$Value,"<utf-16/>" + $Password)).Return
            }
            else
            {
                $SettingResult = ($Interface.SetBIOSSetting($Name,$Value)).Return
            }
            

            if ($SettingResult -eq 0)
            {
                Write-Host "Successfully set $Name to $Value"
                #$Script:SuccessSet++
            }
            else
            {
                Write-Error "Failed to set $Name to $Value"
                #$Script:FailSet++
            }
        }
    }
    #Setting not found
    else
    {
        Write-Warning "Setting $Name not found" 
        #$Script:NotFound++
    }
}

#Set HP BIOS settings - password is not set

ForEach($Setting in $Settings){
    $Data = $Setting.Split(';')
    Set-HPBiosSetting -Name $Data[0].Trim() -Value $Data[1].Trim()
}
```

Install Updated BIOS

```
## Download BIOS
Get-HPBiosUpdates -flash -yes -Bitlocker suspend -quiet -Verbose
```

Initialize TPM

```
Initialize-TPM -AllowClear
```

Test for TPM Present

```
## Test for TPM
if((Get-Tpm).TpmPresent -eq $true) {
    Write-Output 'Enabled'
    Exit 0
} else {
    Write-Output 'Disabled'
    Exit 1
}
```

Test for TPM Ready

```
## Test for TPM
if((Get-Tpm).TpmReady -eq $true) {
    Write-Output 'Enabled'
    Exit 0
} else {
    Write-Output 'Disabled'
    Exit 1
}
```

Run Scheduled Task

```
$TaskName = "Convert-McAfeeToBitlocker"
Get-ScheduledTask -TaskName $TaskName | Start-ScheduledTask
```