---
layout: default
title: "My Enumerate-EsxLunPaths function"
date: 2020-10-06
description: Get-ScsiLun is slow, this way is much faster!
categories: [VMware, ESXi, Storage, PowerCLI, PowerShell]
---

# The fastest way to get all LUN paths in vSphere

The larger your environment gets, the more unwieldy it can become. Ideally, you have good process controls in place so only expert hands are modifying the storage environment, but sometimes mistakes get made even by the most veteran engineers. In worst case scenarios, you have very low level techs with insufficient training and too much pressure to just close tickets without checking their work. If you've never worked in such an environment, pray that you don't! But since I have in the past, I developed the fastest way to quickly check all LUN paths for ESX hosts. 

## Case 1: Needle in a haystack

For whatever reason, you're looking for a few paths that are down and you need to correlate it back to an NAA ID or a datastore. For those of you that have tried to do this through the GUI I can already hear you groaning. This is the kind of thing that you pitch over the wall for the storage guys to track down, or else the FNG on your team. But with this function, you can easily iterate through all your hosts and look for any paths that are not in the desired state.

## Case 2: Daily health report

One time I worked from dawn until dusk for a week straight because of a neglected storage environment. It turns out that if ESXi cannot use a path, it will keep trying to bring the path up. I'm paraphrasing poorly but this was a behavior change between ESXi 5 and ESXi 6. My organization at the time had decided to be rather lazy when it came to deprovisioning LUNs so there were a LOT of orphaned paths out there that our hosts were dutifully trying to reconnect to. It turns out that if this goes on long enough, or you get enough orphaned paths, you can start crashing services! We had hosts dropping into disconnected state randomly.

After that, I figured out that a daily health report, looking for orphaned paths was a way to go. This has become a staple for me. I want to know the state of all paths in my environments at all times. I trust my storage guys but I also like to verify. Plus, the richness of the information provided by this function makes troubleshooting and narrowing it down a breeze.

## Case 3: A jumping-off point

In another case, I had a client environment that was showing periodic storage latency issues. I used this function to rapidly enumerate all LUNs on each host and piped that into `Get-Stat` and created an hourly report to see which LUNs were showing increases in latency and throughput. 

# The Code

Without further ado

```powershell
function Enumerate-EsxLunPaths
    {
    param($vmhost)
    $data = @()
    
    # This is the meat and potatoes here! 
    # These 4 lines are lightning fast and gather all the information you need
    # The rest of the script is just massaging it into an object to return
    $storage_view = Get-View $vmhost.ExtensionData.ConfigManager.StorageSystem
    $luns = $storage_view.StorageDeviceInfo.ScsiLun 
    $multipathdata = $storage_view.StorageDeviceInfo.MultipathInfo.Lun
    $mounts = $storage_view.FileSystemVolumeInfo.MountInfo | Where-Object {$_.Volume.Type -like "vmfs"}
    
    foreach ($mount in $mounts)
        {
        try
            {
            $extent = [string]$mount.Volume.Extent.DiskName
            $lun = $luns | Where-Object {$_.displayname -like "*$extent*"}
            $paths = ($multipathdata | Where-Object {$_.Lun -eq $lun.Key}).Path

            foreach ($path in $paths)
                {

                # This is where I create an object to capture all the information
                $info = "" | Select-Object vol_name,vol_path,vol_uuid,vol_extent,lun_uuid,lun_device,lun_key,lun_name,lun_vendor,path_name,path_state,path_working
                $info.vol_name = $mount.Volume.Name
                $info.vol_path = $mount.MountInfo.Path
                $info.vol_uuid = $mount.Volume.Uuid
                $info.vol_extent = $extent

                $info.lun_uuid = $lun.Uuid
                $info.lun_device = $lun.DeviceName
                $info.lun_key = $lun.Key
                $info.lun_name = $lun.CanonicalName
                $info.lun_vendor = $lun.Vendor

                $info.path_name = $path.Name
                $info.path_state = $path.State
                $info.path_working = $path.IsWorkingPath
                    
                $data += $info
                }
            }
        catch { continue }
        }    
    return $data
    }
```
