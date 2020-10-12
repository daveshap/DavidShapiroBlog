---
layout: default
title: "Monitoring VMware VM and ESXi Throughput"
date: 2020-10-12
description: Network and disk IO can kill your environment
---

As a VMware admin or engineer, I'm sure you know that the first thing people ask about when VM performance is slow is for more CPU and RAM. 
It is true that there are some memory-hungry applications out there and right-sizing for CPU is critical. But these are only two of the four food groups of computing. 
Just as critical are network and storage, and in datacenter environments, both are subject to shared networks in the form of TCP/IP, FC, SAN, and FCoE. By extension, they can be subject to latency and congestion problems.
Most people's experience with computers is with local storage, which is almost never going to be a bottleneck. Even more confounding is the fact that a bottleneck on any one of these resources can appear to impact performance elsewhere.
For instance, if storage is too slow, you might start slamming your CPU and memory as SQL queries pile up. Or if you're short on RAM, you might start swapping to disk, slamming storage speed. 

# VM Throughput Report

Here, I gather usage statistics for a period of 24 hours and find the peak usage. I then pare it down to the top talkers in each category. 
This kind of report is helpful to have on hand in case something changes, you can just glance at the report every morning to see if a new VM is spiking. 
More often than not, this will help you identify problematic VMs before they percolate up to the awareness of management, leadership, and even other teams. 
I cannot tell you how many times I've spotted an errant database or backup server before anyone else noticed it and headed off a high visibility issue. 
An ounce of prevention is worth a pound of cure! The ideal crisis is the one that never happens!

```powershell
$vms = Get-VM | Where-Object {$_.PowerState -like "*on*"} | sort-object
$data = @()


foreach ($vm in $vms)
    {
    $net = $vm | Get-Stat -Stat net.usage.average -Start (Get-Date).AddDays(-1) -Finish (Get-Date) -MaxSamples 5000 | Sort-Object -Property value -Descending | Select-Object -First 1
    $disk = $vm | Get-Stat -Stat disk.usage.average -Start (Get-Date).AddDays(-1) -Finish (Get-Date) -MaxSamples 5000 | Sort-Object -Property value -Descending | Select-Object -First 1
    $info = "" | Select-Object vm,max_net_kbps,max_net_time,max_disk_kbps,max_disk_time
    $info.vm = $vm.Name
    $info.max_net_kbps = $net.Value
    $info.max_net_time = $net.Timestamp
    $info.max_disk_kbps = $disk.value
    $info.max_disk_time = $disk.timestamp
    $info | fl
    $data += $info
    }

Clear-Host

$data | Sort-Object -Property max_net_kbps -Descending | Select-Object vm,max_net_kbps,max_net_time -First 50 | ft 

$data | Sort-Object -Property max_disk_kbps -Descending | Select-Object vm,max_disk_kbps,max_disk_time -First 50 | ft 
```

At the end you can do whatever you want with the data. I send it out via email once a day. Usually, I'm really lazy and just do `Out-String` and set the output into `<pre>` tags in an HTML email. It ain't pretty but it's fast and valuable! 

I also recommend removing the additional console output if you do schedule this as an unattended job. 

# ESX Host Throughput

This is the same exact information just with a different focus. 

```powershell
$vmhosts = Get-VMHost | Where-Object {$_.ConnectionState -like 'connected'} | Sort-Object

$data = @()

foreach ($vmhost in $vmhosts)
    {
    $net = $vmhost | Get-Stat -Stat net.usage.average -Start (Get-Date).AddDays(-1) -Finish (Get-Date) -MaxSamples 5000 | Sort-Object -Property value -Descending | Select-Object -First 1
    $disk = $vmhost | Get-Stat -Stat disk.usage.average -Start (Get-Date).AddDays(-1) -Finish (Get-Date) -MaxSamples 5000 | Sort-Object -Property value -Descending | Select-Object -First 1
    $info = "" | Select-Object vmhost,max_net_kbps,max_net_time,max_disk_kbps,max_disk_time
    $info.vmhost = $vmhost.Name
    $info.max_net_kbps = $net.Value
    $info.max_net_time = $net.Timestamp
    $info.max_disk_kbps = $disk.value
    $info.max_disk_time = $disk.timestamp
    $info | fl
    $data += $info
    }

Clear-Host

$data | Sort-Object -Property max_net_kbps -Descending | Select-Object vmhost,max_net_kbps,max_net_time -First 20 | ft 

$data | Sort-Object -Property max_disk_kbps -Descending | Select-Object vmhost,max_disk_kbps,max_disk_time -First 20 | ft
```

You can add other helpful information such as Cluster if you like. 

# Datastore Latency

Storage latency is one of those things that is horribly counter-intuitive to pretty much everyone except virtualization and storage folks, and some cross-pollinated network folks. 
App folks, developers, and DBAs tend to not grasp this problem unless their domain deals specifically with storage technologies. This is nothing against them - we all have domains of expertise for a reason.
Datastore latency is one of those more arcane metrics that is harder to get but incredibly critical when you need it. Hence this script!

IMPORTANT: This script relies upon another function I have documented here: [Enumerate-EsxLunPaths](https://daveshap.github.io/DavidShapiroBlog/2020/10/06/enumerate-esxlunpaths.html)

```powershell
$stat_list = "disk.deviceReadLatency.average","disk.deviceWriteLatency.average"
$host_data = @()

foreach ($cluster in Get-Cluster)
    {
    $vmhosts = $cluster | Get-VMHost | Where-Object {$_.connectionstate -like "connected"}
    foreach ($vmhost in $vmhosts)
        {
        $stats = $vmhost | Get-Stat -Realtime -Stat $stat_list
        $stats = $stats | Where-Object {$_.Value -gt 200}  # change this threshold to squelch noise
        if ($stats -eq $null) { continue }
        $lun_data = Enumerate-EsxLunPaths $vmhost
        foreach ($stat in $stats)
            {
            $lun = $lun_data | Where-Object {$_.lun_name -like $stat.Instance} | Select-Object -First 1
            $info = "" | Select-Object host,cluster,max_latency_ms_15_min,datastore,device_id
            $info.host = $vmhost.Name
            $info.cluster = $cluster.name
            $info.max_latency_ms_15_min = $stat.value
            $info.datastore = $lun.vol_name
            $info.device_id = $stat.instance
            $info | fl
            $host_data += $info
            }
        }
    }
```

The biggest problem with this set of stats is that they expire very quickly so you may need to set this to run repeatedly, regularly checking for problematic LUN paths. 
If you've ever had a major issue with storage latency, I'm sure you're already salivating. This script relies upon my `Enumerate-EsxLunPaths` function, which is lightning fast. 
You can run this script on demand or as a scheduled task. Generally, I have used it as an on-demand tool to help troubleshoot big issues while on calls real-time. 
You're most likely to need this particular gem when backup jobs are taking too long or Oracle queries are bogging down. 

# BONUS: UCS Statistics one-liner!

Okay, so you know how VMware basically nests the OSI model inside the OSI model? Cisco UCS takes that to the *nth* degree by abstracting all storage and networking over the IOM uplinks. 
I once discovered just how crazy this is when a network engineer discovered one particular IOM port was saturated during backup jobs. I traced it down and discovered that `Chassis/FEX Discovery Policy` was not set to `Port Channel`, which meant that every other server was pinned to a given IOM uplink on one side. Yikes!
It had taken a few weeks of complaints getting louder and louder until we discovered that **storage**-intensive backup jobs were trashing **network throughput** because of this shared nature of UCS. 

Without further ado, the magical command that can enumerate more UCS statistics than you ever wanted!

```powershell
Get-UcsStatistics -Current | Where-Object {$_.Rn -like "tx-stats"}
```
