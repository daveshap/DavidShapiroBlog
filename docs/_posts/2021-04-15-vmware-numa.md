---
layout: default
title: "Overprovisioned VMs cause CPU contention due to NUMA boundaries"
date: 2021-04-15
description: CPU READY will kill your performance even if CPU usage is low
categories: [VMware, KB, NUMA]
---

# Symptoms

1. VMs are inexplicably slow, even with low overall CPU usage at the ESXi host level or VM level
2. CPU READY times are high on the GUI charts or ESXTOP
3. VMs have high vCPU count (overprovisioned)
4. \[Optional\] Overprovisioned VMs reside on same ESXi host

# Cause

When a VM has more vCPU cores allocated than exist in a single CPU on its underlying host, it will have to cross **NUMA boundaries** when performing CPU instructions.  See below graphic for example.

![NUMA Nodes](/assets/numa.png)

- A smaller VM will fit nicely within a single NUMA boundary
- A larger VM will span NUMA boundaries

This situation can cripple the performance of either or both VMs. The overprovisioned VM can steal resources from the correctly provisioned VM. 

# Resolution

## Reduce vCPU count

If possible, reduce the vCPU count of the overprovisioned VM. I have provided a script (below) to help assess the correct size for all VMs in your environment. Right-sizing your VMs is best practice, and it can prevent many problems, plus it can increase performance. 

```powershell
Connect-VIServer $vcenter_fqdn  # change this to your vCenter name!
$vms = Get-VM | Where-Object {$_.PowerState -like "*on*" -and $_.NumCpu -ge 4}  # Most VMs with fewer than 4vCPU are boring
$data = @()

foreach ($vm in $vms)
    {
    $ready = $vm | Get-Stat -Stat cpu.ready.summation -Start (Get-Date).AddDays(-365)
    $ghz = $vm | Get-Stat -Stat cpu.usagemhz.average -Start (Get-Date).AddDays(-365)
    $info = "" | select VM,vCPU,GhzUsed,ReadySecPerDay,GhzCapacity,NeededCores,HostSockets,HostCores,NumaCores
    $info.VM = $vm.name
    $info.vCPU = $vm.NumCpu
    $info.ReadySecPerDay = [int](($ready | Measure-Object -Property Value -Average).Average / 1000)
    $info.GhzUsed = [math]::Round((($ghz | Measure-Object -Property Value -Average).Average / 1000), 2)
    $info.GhzCapacity = $vm.NumCpu * 2.1
    $info.NeededCores = [math]::ceiling(($ghz | Measure-Object -Property Value -Average).Average / 2100) * 2
    $info.HostSockets = $vm.VMHost.ExtensionData.Hardware.CpuInfo.NumCpuPackages
    $info.HostCores = $vm.VMHost.ExtensionData.Hardware.CpuInfo.NumCpuCores
    $info.NumaCores = $vm.VMHost.ExtensionData.Hardware.CpuInfo.NumCpuCores / $vm.VMHost.ExtensionData.Hardware.CpuInfo.NumCpuPackages
    $data += $info
    $info | fl
    }


Clear-Host
$data | ft
$data | Export-Csv all_vm_cpu_ghz_ready.csv -NoTypeInformation
```

Here's an example of the output from this script:

```
<output coming soon!>
```

Explanation of fields:

| Field | Description |
|---|---|
| VM | Name of the VM |
| vCPU | Count of vCPU cores  assigned to the VM |
| GhzUsed | Average Ghz consumed by the VM over the past year |
| ReadySecPerDay | Average number of total seconds the VM's cores were in "CPU READY" state (waiting for CPU time from the host) |
| NeededCores | Double the number of cores required to accomodate average load, assuming a core provides about 2Ghz |
| HostSockets | Number of physical processors installed in the ESXi host |
| HostCores | Total number of cores between all sockets |
| NumaCores | Number of cores available to each NUMA node | 

It's normal to have some CPU READY time in your virtual environment. It is, after all, a shared environment. Many VMs will average less than 200 seconds of CPU READY per day. However, when you start to get larger and larger vCPU counts, you can see CPU READY piling up into the tens of thousands. This is all performance that is just out the window. Right-sizing all your VMs will prevent them from competing for resources.

## Change the vCPU settings

In the cases where you need more vCPU cores than exist on a given NUMA node, you can fiddle with the CPU cores/sockets settings in the VM virtual hardware settings. This VMware blog has far more detail about it: [https://blogs.vmware.com/performance/2017/03/virtual-machine-vcpu-and-vnuma-rightsizing-rules-of-thumb.html](https://blogs.vmware.com/performance/2017/03/virtual-machine-vcpu-and-vnuma-rightsizing-rules-of-thumb.html)

## Move the VMs to hosts with higher core counts

Lets say you have some hosts with 2x10 cores but they have VMs with 16 cores. You could move those VMs to larger hosts with 2x16 cores or more. 

## Use CPU reservations

Important note: This just tells VMware to rob Peter to pay Paul. You can use this to ensure that production VMs get resources when push comes to shove and that non-prod VMs get crippled. 

# Conclusion

The easiest thing to do is simply right-size your VMs. Correctly allocated CPU and RAM will prevent this problem, as well as many others. 









