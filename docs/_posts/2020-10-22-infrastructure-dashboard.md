---
layout: default
title: "My Infrastructure Dashboard"
date: 2020-10-22
description: When you're responsible for dozens of systems and hundreds of hosts...
categories: [VMware, UCS, Python, PowerShell, Flask, Monitoring, Dashboard]
---

# Enterprise Tools are Expensive

I'm not gonna knock tools like vRops and UCS Director and SolarWinds. They are awesome tools, but they are expensive. 
They also tend to offer a lot of functionality that I might not need. Lastly, they might not integrate with all my systems. Here's a summary of what I want in a dashboard:

- Single pane of glass for EVERYTHING I'm responsible for
- Inventory of hardware, servers, switches, blades, etc
- All active faults, alarms, etc, across all my systems
- Capacity and throughput for all my systems
- I want it to be FAST and CLEAN

Basically, I want the ability to glance at one thing to quickly ascertain the state of my entire environment. 
I set up a previous version of this at a while back and several of my team members found it indispensible - a great way to keep dynamic track of all inventory.
This is especially critical when you have blades, chassis, and vCenters that don't necessarily all talk to each other. 

My first iteration of this was something like 7 years ago, when I tried to make a PowerShell GUI app that I called "vCommander". 
It didn't go too far because I didn't have clarity of purpose. I thought I wanted a fast way to run all my scripts, but for that I just use PowerShell ISE and now Rundeck.
[Rundeck](https://www.rundeck.com/) provides all the script-via-web functionality you could ever want and it's open source!

# Python Flask

Ever build a web page from scratch? Well, now you can! With Python and Flask! Personally, I've created a few utility tools and REST APIs with Flask. 
It's one of my favorite little things. 

## Bare Bones Version

```python
import flask
import json


def json_to_html_table(data):
    # data must be list of dicts, where all dicts have same keys
    result = '<table><tr>'
    keys = data[0].keys()
    for k in keys:
        result += '<th>%s</th>' % k
    for row in data:
        result += '<tr>'
        for k in keys:
            try:
                if 'http' in row[k]:
                    short_name = row[k].replace('https://','').replace('http://', '').split('.')[0]
                    result += '<td><a href="%s" target="_blank">%s</a></td>' % (row[k], short_name)
                else:
                    result += '<td>%s</td>' % row[k]
            except:
                result += '<td>%s</td>' % row[k]
        result += '</tr>'
    result += '</table>'
    return result


def load_json_content(filepath):
    html = ''
    with open(filepath, 'r') as infile:
        content = json.load(infile)
    for section in content:
        html += '\n<div>\n<h2>%s</h2>\n' % section['Header']
        html += json_to_html_table(section['Data'])
        html += '\n</div>'
    return html


app = flask.Flask('InfrastructureDashboard')


@app.route('/', methods=['get'])
def home():
    html = base_html
    html += '\n<h1>Fast Links</h1>\n<div class="flex-container">'
    html += load_json_content(<path to home content>)
    html += '\n</div>\n</body>\n</html>'
    return html

# add more routes here! 

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=443, ssl_context='adhoc')
```

## JSON Content

My `home.json` file looks like this:

```json
[
	{"Data": [
		{"Type": "vCenter", "Label": "Prod", "Link": "<url to prod VC>"},
		{"Type": "UCS", "Label": "Prod", "Link": "<url to prod UCS"}
		],
	"Header": "Site 1"},

	{"Data": [
		{"Type": "vCenter", "Label": "Prod", "Link": "<url to prod VC>"},
		{"Type": "UCS", "Label": "Prod", "Link": "<url to prod UCS"}
		],
	"Header": "Site 2"}
]
```

*Critical:* You can populate `Data` with whatever you want! This can be throughput data, faults, alarms, blade and host inventory, VMs, datastores, switches - anything! 

## Base HTML

The "Base HTML" content is actually inspired by Jekyll, which this site uses. Basically, my infrastructure dashboard interprets JSON in real-time instead of generated markdown. 

```html
<html>
<head>
<title>Infrastructure Dashboard</title>
<style>
* {font-family: Calibri;}
a {color: #3366CC;}
table {border-width: 1px; border-style: solid; border-color: lightgray; border-collapse: collapse; white-space: nowrap;}    
th {border-width: 1px; padding: 4px; border-style: solid; border-color: lightgray; background-color: lightskyblue; white-space: nowrap;}
td {border-width: 1px; padding: 4px; border-style: solid; border-color: lightgray; white-space: nowrap;}
.flex-container {display: flex; flex-wrap: wrap; background-color: LightGray;}
.flex-container > div {margin: 10px; background-color: White; padding: 10px}
</style>
</head>
<body>
<a href="/">Home</a>&nbsp;&mdash;&nbsp;
<a href="/clustercapacity">vSphere Cluster Capacity</a>&nbsp;&mdash;&nbsp;
<a href="/ucsfaults">UCS Faults</a>&nbsp;&mdash;&nbsp;
<a href="/ucsthroughput">UCS Throughput</a>
```

This "base HTML" is used to instantiate all pages returned, so they always include the style and navigation. I also make use of flex containers, because I just discovered them and they are awesome.

And that's basically it. You can add endpoints all day long, you just need JSON files to pull from. 

# Where to get your data

## Homepage aka "Fask Links" page

My homepage is statically coded. It contains a list of all my web-based tools and services. That includes vCenter, UCS, and other such stuff. I've got mine organized by site.
You could also organize yours by type of service/device. You can also change the purpose entirely. This is just where I started as I wanted one convenient landing page with links
to all my servers and services. I set this as my browser homepage. 

## Report Data

I've written plenty of other posts about monitoring and throughput gathering, so I'm going to assume you have seen those or have your own data collection scripts running.
There are many ways to skin that cat, so I'll just add a tidbit about how I dump my reports to JSON in PowerShell:

```powershell
$json_path = <filepath to desired JSON file>
Remove-Item -Confirm:$false -Force -Path $json_path -ErrorAction SilentlyContinue
$report_data | ConvertTo-Json -Depth 5 | Out-String | Set-Content -Path $json_path
```

The `-Depth 5` parameter is important as the default depth for `ConvertTo-Json` is only 2. This drops pretty JSON files wherever you want them, which can then be converted to nice HTML tables. 
I make sure to sort and massage my data so that it is clean and readable. Some ideas for you:

- Critical alerts page
- Saturated links page
- Full datastores page
- VM inventory page
- ESX host inventory page
- Blade/chassis inventory page
- Network/disk latency page

Basically, you can slice and dice your environment and data any way you like!

# Authentication

A previous version of this app had authentication. I might revisit that and implement some sort of ACL or offload authentication to LDAP/AD or even vSphere. 
I'm not too worried because this information is read-only and, depending on where you deploy it, is available only inside your management networks. 



