---
layout: default
title: "Monitoring Cisco HyperFlex REST API with Python"
date: 2020-10-16
description: I'm actually super salty about this
---

# The REST API Explorer

I judge companies and products harshly by their documentation. API documentation is especially stringent because, well, an API is meant to be standardized. There's no excuse for sloppy REST API documentation.

Cisco HyperFlex documentation... [leaves a lot to be desired](https://developer.cisco.com/docs/ucs-dev-center-hyperflex/#!connecting-to-the-hyperflex-rest-api-explorer). Seriously, just compare that to [Twitter's API documentation](https://developer.twitter.com/en/docs/twitter-api). It's pitiful. 

To make matters worse, the HyperFlex REST API apparently changes drastically from release to release so it's increasingly critical that you get familiar with the REST API Explorer. There is something to be said for having a robust API explorer, so they get some points back for that one. Still, you can see that I have a lot of debug output in these scripts, and I decided to keep them to show you just much effort it was to figure this out!

# Authentication

This is what I got to work after cobbling together some snippets from around the internet and gaining access to my own instance of HyperFlex. A few things to note: The `client_id` and `client_secret` are apparently mandatory but not really documented anywhere that I recall.

```python
import requests
import json

def auth_to_hx(fqdn, username, password):
    url = 'https://%s/aaa/v1/auth?grant_type=password' % fqdn
    headers={'content-type':'application/json'}
    payload = {'username': username,
        'password': password,
        'client_id': 'HxGuiClient',
        'client_secret': 'Sunnyvale',  # this is the default, you can change it 
        'redirect_uri': 'http://%s' % fqdn}
    try:
        response = requests.post(url,headers=headers,data=json.dumps(payload),verify=False,timeout=40)
        if response.status_code == 201:
            #print('Login succeeded to', fqdn)
            return response.json()  # this is your authentication token
        else:
            #print('LOGIN FAILED', fqdn.upper())
            #print(response.status_code)
            #print(response.text)
            return None
    except Exception as oops:
        #print('LOGIN FAILED', fqdn.upper())
        #print(oops)
        return None
```

# Enumerate Clusters

HyperFlex is organized in a few ways. There are the Clusters and the Platform. We will get to the Platform in a moment. The Clusters are the meat and potatoes, though. Pass the `token` you get from authenticating to this function.

Note: The `timeout` option became necessary because sometimes HyperFlex doesn't respond as fast as you'd like. Sometimes it's lightning fast. I don't really know why there's variance. 

Another note: The `cluster` object returns has child elements `/entityRef/id`. This is the `Cluster UUID` or `CUUID` that you will need to reference the cluster by later. Again - this was not documented anywhere! To make matters even worse, the CUUID is URL encoded so you have to manually convert it back like so: `cuuid = cluster['entityRef']['id'].replace(':','%3A')`. Starting to see why I'm grumpy about the HyperFlex REST API?

```python
def get_hx_clusters(fqdn, token):
    url = 'https://%s/rest/clusters' % fqdn
    #print(url)
    headers={'content-type':'application/json','Authorization':token['token_type'] + token['access_token']}
    response = requests.get(url,headers=headers,verify=False,timeout=40)
    if response.status_code == 200:
        return response.json()
    else:
        #print(response.status_code, response.text)
        return None
```

# Get Platform Alarms

The Platform is the management/controller portion. As best I can figure, this is roughly analogous to vCenter in vSphere. It's similar to the methodology used in other hyperconverged systems, such as HPE Simplivity, where you've got a VM embedded that runs the software and manages the backplane. In UCS-world, the closest thing is likely UCSM that runs inside the Fabric Interconnects. 

The bottom line is that you will want to monitor the HyperFlex platform as well as the clusters/hosts. Most of the time there's going to be nothing reported, but that's okay. Still, I'd much prefer the old fashioned `Get-UcsFault` style. One request, all alarms. Period, end of story. Even VMware doesn't have anything that good. Alas, it seems like we were lucky only the once. Maybe I'll combine all these functions into a single `Get-HxFault`?

The platform layer will return alarms dealing with things like the manager configuration, backups, and the like. 

```python
def get_hx_alarms(fqdn, token):
    url = 'https://%s/rest/virtplatform/alarms' % fqdn
    #print(url)
    headers={'content-type':'application/json','Authorization':token['token_type'] + token['access_token']}
    response = requests.get(url,headers=headers,verify=False,timeout=40)
    if response.status_code == 200:
        return response.json()
    else:
        #print(response.status_code, response.text)
        return None
```

I actually don't know what these might look like because I didn't record the one I had, I just fixed it. If I recall correctly it was something like "email alerts not configured" or "phone home support unreachable". Those kinds of things. 

# Get Cluster Alarms

Cluster alarms are separate from platform alarms, which are yet still separate from cluster health. I really wish it weren't so, but it is what it is. Cisco did a great thing by giving you all UCS faults in one endpoing via `Get-UcsFault` and I will forever be salty that no other platforms seem to work this way. I just want to monitor everything and fix things before they blow up. Why is that so hard? 

Okay, I'll quit whining. Here's the cluster alarms function. This will return things like host and VM level errors. 

```python
def get_hx_cluster_alarms(fqdn, token, cuuid):
    url = 'https://%s/coreapi/v1/clusters/%s/alarms' % (fqdn, cuuid)
    #print(url)
    headers={'content-type':'application/json','Authorization':token['token_type'] + token['access_token']}
    response = requests.get(url,headers=headers,verify=False,timeout=40)
    if response.status_code == 200:
        return response.json()
    else:
        print(response.status_code, response.text)
        return None
```        

This is what a cluster alarm might look like:

```json
[ { 'acknowledged': False,
    'acknowledgedTime': 0,
    'acknowledgedTimeAsUTC': '',
    'description': 'Default alarm to monitor virtual machine memory usage',
    'entityName': '<vm name>',
    'entityType': 'VIRTUALMACHINE',
    'entityUuId': 'vm-1111',
    'message': 'Default alarm to monitor virtual machine memory usage',
    'name': 'alarm-6.vm-1111',
    'status': 'CRITICAL',
    'triggeredTime': 1002709437316,
    'triggeredTimeAsUTC': '2010-07-16T00:50:37Z',
    'uuid': 'alarm-6!!Alarm!!alarm-6!!vm-3517!!VirtualMachine!!vm-3517'}]
```    


# Get Cluster Health

Cluster health is fun. This primarily focuses on the storage replication status. 

```python
def get_hx_cluster_health(fqdn, token, cuuid):
    url = 'https://%s/coreapi/v1/clusters/%s/health' % (fqdn, cuuid)
    print(url)
    headers={'content-type':'application/json','Authorization':token['token_type'] + token['access_token']}
    response = requests.get(url,headers=headers,verify=False,timeout=40)
    if response.status_code == 200:
        return response.json()
    else:
        print(response.status_code, response.text)
        return None
```

This is what it might look like:

```json
{ 'dataReplicationCompliance': 'COMPLIANT',
  'resiliencyDetails': { 'dataReplicationFactor': 'TWO_COPIES',
                         'hddFailuresTolerable': 1,
                         'messages': ['Storage cluster is healthy. '],
                         'nodeFailuresTolerable': 1,
                         'policyCompliance': 'COMPLIANT',
                         'resiliencyState': 'HEALTHY',
                         'ssdFailuresTolerable': 1},
  'state': 'ONLINE',
  'uuid': '<cuuid>',
  'zkHealth': 'ONLINE',
  'zoneResiliencyList': []}
```

