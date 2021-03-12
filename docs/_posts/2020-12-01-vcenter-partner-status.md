---
layout: default
title: "Quick way to check vCenter Linked Mode replication status"
date: 2020-12-01
description: Could not connect to one or more vCenter Server systems... :443/sdk
categories: [VMware, KB]
---

This will be a very quick KB article style blog post.

# Symptoms

- You see a banner that reads `Could not connect to one or more vCenter Server systems: <VC FQDN>:443/sdk` when logging into linked-mode vCenter
- You may be missing one or more vCenters from the inventory tree, depending on which vCenter you log into

# Possible Causes

- vCenter services failing
- Disk space low
- Network/firewall blocking communication
- Platform Services Controller misconfiguration: [VMware KB 2050273](https://kb.vmware.com/s/article/2050273)

You can check on VAMI (:5480) to verify health of services, database, and disk space.

# Diagnosis

Here are some commands you can run to see what vCenter is doing behind the scenes!

- `/usr/lib/vmware-vmdir/bin/vdcrepadmin -f showpartnerstatus -h localhost -u administrator`
- `/usr/lib/vmware-vmdir/bin/vdcadmintool` >> option 6
- Check in VPXD logs
- Check ping/telnet
- More documentation here: [VMware KB 2127057](https://kb.vmware.com/s/article/2127057)
