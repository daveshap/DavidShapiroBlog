---
layout: default
title: "Round Robin IOPS settings in ESXi"
date: 2020-10-01
description: Scripting a quick change to default IOPS Round Robin settings in ESXi
---

## Why do you need to change default IOPS settings?

Some storage vendors or SAN arrays expect different configurations. One example is the IOPS Round Robin limit. By default, ESXi uses 1000. Basically, what this means is that ESXi will send 1000 IOPS down each path before switching to the next when using Round Robin. For faster storage we often need to set this much lower, such as to 1 or 10. With things like all-flash arrays, the storage controller can handle IOPS a lot faster than before, so you can get much better performance this way. Always follow your vendor's best practices, though. 

## The script

For the sake of simplicity, I'm only going to show you the meat and potatoes of my scripts. I'm going to assume you understand the basics, like connecting to vCenter and such.

```powershell
foreach ($vmhost in $vmhosts)
    {
    $cli = $vmhost | Get-EsxCli -V2
	
	# Get your LUNs and filter out any that you don't want to modify
    $devices = $cli.storage.nmp.device.list.Invoke() | Where-Object {$_.DeviceDisplayName -like "<your filter here>" -and $_.pathselectionpolicy -like 'vmw_psp_rr'}
    foreach ($device in $devices)
        {
		# Create an argument's list to invoke against the ESXCLI
        $d = @{}
        $d['iops'] = 1
        $d['device'] = $device.Device
        $d['type'] = 'iops'
        $result = $cli.storage.nmp.psp.roundrobin.deviceconfig.set.Invoke($d)
        Write-Host $vmhost.name $device.device $result
        }
    }
```

### esxcli

I discovered ESXCLI years ago. It's great. It gives you access to the deepest levers and buttons on ESXi but over remote. No SSH or anything necessary. The only wonky part that takes some getting used to is creating the arguments. There's more than one way to skin the cat, so please don't judge my clunky method too harshly. The key takeaway is that you compose the argument in the form of a hashtable and then send it with the "invoke" method.

### devices

Your devices should have some naming convention or other identifying thing. In most cases, the `DeviceDisplayName` will include the vendor name of the LUN. This can be a great way to filter out LUNs that you don't want to monkey with. Generally speaking, though, you can probably just use the `vmw_psp_rr` filter. You can assume that any LUN with Round Robin might be affected. Though you might have multiple arrays on the backend. Do whatever you want here!
