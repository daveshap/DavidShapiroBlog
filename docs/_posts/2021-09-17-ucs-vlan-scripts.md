---
layout: default
title: "Some helpful UCS VLAN scripts"
date: 2021-09-17
description: Because adding VLANs by hand is hell (and dumb)
categories: [UCS, VLAN, PowerShell]
---

# Export VLANs (from existing domain)

This first script will group all the VLANs and VLAN groups. This is handy if you need to set up a new UCS domain based on an existing one. Plus it dumps everything to CSV so you can double check and/or update before moving on.

```powershell
$ucsname = "UPDATEME"
if ($ucs_creds -eq $null) { $ucs_creds = Get-Credential -Message "Format = ucs-DOMAIN\username"}

if ($DefaultUcs -ne $null) { Disconnect-Ucs -ErrorAction Stop }
Connect-Ucs -Name $ucsname -Credential $ucs_creds -ErrorAction Stop

# export VLAN list
Get-UcsVlan | Select-Object Id,Name | Export-Csv -NoTypeInformation -Force -Path "$ucsname_VLANS.csv"

# get groups
$groups = Get-UcsFabricNetGroup
$data = @()
foreach ($group in $groups)
    {
    $members = $group | Get-UcsFabricPooledVlan
    foreach ($member in $members)
        {
        $info = "" | Select-Object Group,Vlan
        $info.Group = $group.Name
        $info.Vlan = $member.Name
        $data += $info
        }
    }

$data | Export-Csv -NoTypeInformation -Force -Path "$ucsname_groups.csv"
```

# Add VLANs and Groups

This script presumes that you've already created the groups by hand or with another script. You could easily add them yourself with `Add-UcsFabricNetGroup`.

```powershell
$ucsname = "UPDATEME"
if ($ucs_creds -eq $null) { $ucs_creds = Get-Credential -Message "Format = ucs-DOMAIN\username"}

if ($DefaultUcs -ne $null) { Disconnect-Ucs -ErrorAction Stop }
Connect-Ucs -Name $ucsname -Credential $ucs_creds -ErrorAction Stop

$vlans = Import-Csv -Path "$ucsname_VLANS.csv"
$groups = Import-Csv -Path "$ucsname_groups.csv"

$cloud = Get-UcsLanCloud 

foreach ($vlan in $vlans)
    {
    Write-Host "Adding VLAN" $vlan.id $vlan.name
    $cloud | Add-UcsVlan -CompressionType included -DefaultNet no -Id $vlan.Id -McastPolicyName "" -Name $vlan.Name -PolicyOwner local -PubNwName "" -Sharing none -ErrorAction Continue
    }

foreach ($vlan in $groups)
    {
    write-host "Adding VLAN to group" $vlan.Vlan $vlan.Group
    $group = $cloud | Get-UcsFabricNetGroup -Name $vlan.Group
    $group | Add-UcsFabricPooledVlan -ModifyPresent -Name $vlan.Vlan -ErrorAction Continue
    }
```

# BONUS: Add vNIC Template one-liner

I haven't figured out how to add VLAN groups to vNIC templates yet but this will still help you provision a boadload of vNIC templates.

```powershell
Get-UcsOrg | Add-UcsVnicTemplate -Name "NAME" -Descr "DESCRIPTION" -PolicyOwner local -Mtu 1500 -PinToGroupName "PIN_GROUP" -CdnSource vnic-name -TemplType updating-template -RedundancyPairType none -IdentPoolName "MAC_POOL" -SwitchId A
```

