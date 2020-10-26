---
layout: default
title: "Infrastructure Dashboard Update"
date: 2020-10-26
description: Now with cookies!
categories: [VMware, UCS, Python, PowerShell, Flask, Monitoring, Dashboard]
---

# Agile

I am a big fan of Agile. This is basically how I live my whole life. When I was younger, my motto was *just wing it*. As I gained experience, I learned to trust established methods 
but the ability to improvise remains. Because of this, you can expect to see rapid iterations from pretty much any of my projects. Agile really is just a natural way of doing things, 
although I will concede that it doesn't lend itself to pass/fail projects like building skyscrapers. 

# Security

I've played with various ways of securing my dashboard in the past. One method was to offload RBAC to vSphere. The assumption was this: If you've got certain privileges in vSPhere,
which talks to Active Directory, you've probably got at least read-only privileges for things like VMs and blades. This is probably the better way, but for a small, fast tool, 
I also like standalone ability. You should be able to authenticate locally or via LDAP. So let's look at how I do this:

```python
import uuid

sessions = list()

valid_tokens = ['<uuid v4 token>, '<another token>']


def check_session(cookie, ip):
    for session in sessions:
        if session['uuid'] == cookie and session['ip'] == ip:
            return True
    return False


@app.route('/login', methods=['get', 'post'])
def login():
    valid_session = check_session(flask.request.cookies.get('uuid'), flask.request.remote_addr)
    if valid_session:
        return flask.redirect('%s/home' % base_url, code=302)
    if flask.request.method == 'GET':
        html = base_html
        html += '<h1>Infrastructure Dashboard</h1><form action="%s/login" method="post"><input type="text" placeholder="Token" name="token"><button type="submit">Login</button></form>' % base_url
        return html
    elif flask.request.method == 'POST':
        token = flask.request.form['token']
        if token in valid_tokens:
            cookie = str(uuid.uuid4())
            session = {'uuid': cookie, 'ip': str(flask.request.remote_addr)}
            sessions.append(session)
            response = flask.make_response(flask.redirect(base_url))
            response.set_cookie('uuid', cookie)
            return response
        else:
            html = base_html
            html += '<h1>Infrastructure Dashboard</h1><p>Token not accepted</p><form action="%s/login" method="post"><input type="text" placeholder="Token" name="token"><button type="submit">Login</button></form>' % base_url
            return html

@app.route('/<endpoint>', methods=['get'])
def generic_endpoint(endpoint):
    valid_session = check_session(flask.request.cookies.get('uuid'), flask.request.remote_addr)
    if not valid_session:
        return flask.redirect('%s/login' % base_url, code=302)
    # otherwise, keep going
```

Here, I maintain a list of accepted tokens as well as validated sessions. The valid sessions list contains a list of dictionaries containing a UUID and IP address of a host. 
Within the realm of security, authentication, and identity management there is this concept of *three factor authentication*:

1. Something you know
2. Something you have
3. Something you are

The most common expression of this today is when you get a security code texted or emailed to you after already providing a username and password. 
I attempt to do some of the same things here, albeit with far less sophistication. In order to authenticate, you simply need to give me a UUID token. By virtue of length and complexity,
a UUID password is pretty darn strong. It's 128 bits and is nothing but pure entropy. So if you know the secret UUID to gain access, that's a pretty strong indicator that you belong. 

Flask also gives us the ability to set cookies and detect the client address. So for subsequent authentications, I simply check if you have a cookie called `uuid` that contains a uniquely generated UUID just for you.
Your unique UUID is paired with your IP address. My server retains the list of valid sessions internally. 

1. You know the secret UUID
2. You have your personal UUID token
3. You are using a computer with a specific IP address

The IP address would be possible to spoof, but the chances of guessing the UUID paired to that IP address are vanishingly small. I'm sure there are ways to defeat this, but it's a helluva lot stronger than `admin` and `password`!

# Dynamic Content

Okay don't get too excited. This is not interactive content, but rather just cleaner code. The rule of thumb is that if you write something more than once, create a function for it. Do not duplicate code!

```python
import flask
import json
import os
from datetime import datetime
import uuid


base_html = """
<html>
<head>
<title>Dashboard</title>
<style>
* {font-family: Calibri;}
a {color: #3366CC;}
table {border-width: 1px; border-style: solid; border-color: lightgray; border-collapse: collapse; white-space: nowrap;}    
th {border-width: 1px; padding: 4px; border-style: solid; border-color: lightgray; background-color: lightskyblue; white-space: nowrap;}
td {border-width: 1px; padding: 4px; border-style: solid; border-color: lightgray; white-space: nowrap;}
.flex-container {display: flex; flex-wrap: wrap; background-color: LightGray;}
.flex-container > div {margin: 10px; background-color: White; padding: 10px}
tr:nth-child(even) {background-color: #E7F5FE;}
</style>
</head>
<body>
"""

# Yes, I know, I should load the HTML from a file, I'll get around to that... eventually!

base_url = 'https://<yourserver>'
root_dir = '<your root dir>'

endpoints = {
'home': 'Home',
'vmware': 'VMware',
'ucs': 'UCS',
}

def generate_nav_bar(nav):  # nav is the same as endpoints here
    html = ''
    for key in nav.keys():
        html += '<a href="%s/%s">%s</a>&nbsp;&mdash;&nbsp;' % (base_url, key, nav[key])
    return html


def file_timestamp_str(filepath):
    modified = os.path.getmtime(filepath)
    time = str(datetime.fromtimestamp(modified))
    return time
    

@app.route('/<endpoint>', methods=['get'])
def generic_endpoint(endpoint):
    # ...authentication bits... (see above)
    if endpoint == 'favicon.ico':
        return flask.send_from_directory(root_dir, 'favicon.ico', mimetype='image/vnd.microsoft.icon')
    if endpoint not in endpoints.keys():
        return 'Endpoint %s not available' % endpoint, 404
    html = base_html
    html += generate_nav_bar(endpoints)
    try:
        html += '\n<h1>%s</h1>\n<p>Last Updated: %s</p>\n<div class="flex-container">' % (endpoints[endpoint], file_timestamp_str('%s/%s.json' % (root_dir, endpoint)))
        html += load_json_content('c:/fast_links/%s.json' % endpoint)
    except Exception as oops:
        return 'Error loading JSON data: ' + str(oops), 500
    html += '\n</div>\n</body>\n</html>'
    html = html.replace('<title>Dashboard</title>','<title>Dashboard - %s</title>' % endpoints[endpoint])
    return html

```

Here, I dynamically generate the navbar. This allows me to consolidate on a single function for most content. Now I have a `/` which just redirects, 
a `/login` which does exactly what it says, and a `/<endpoint>` which automatically populates whatever the content you ask for. I thought of another way to automate this further,
which was to just enumerate all JSON files in a `data_dir`, but I haven't gotten around to that yet. In that case, I would probably set the filename as the page header/title (minus the .json), 
and then lower-case and remove whitespace for the URL slug. 

Another thing I do here is inject the file modified timestamp. I realized that looking at page after page of HTML tables with no other context was a bit difficult. 
I have separate emails notifying me of when my data is updated, but the web page has nothing. 

# Ideas for endpoints

Here are some types of content I'm exploring for my personal instance:

- Hosts and blades
- Chassis and interconnects
- Serial numbers
- Alarms, faults, and errors
- Storage and network throughput
- Snapshots
- Cluster capacity
- Storage paths

# The Whole Thing

```python
import flask
import json
import os
from datetime import datetime
import uuid



base_html = """
<html>
<head>
<title>Dashboard</title>
<style>
* {font-family: Calibri;}
a {color: #3366CC;}
table {border-width: 1px; border-style: solid; border-color: lightgray; border-collapse: collapse; white-space: nowrap;}    
th {border-width: 1px; padding: 4px; border-style: solid; border-color: lightgray; background-color: lightskyblue; white-space: nowrap;}
td {border-width: 1px; padding: 4px; border-style: solid; border-color: lightgray; white-space: nowrap;}
.flex-container {display: flex; flex-wrap: wrap; background-color: LightGray;}
.flex-container > div {margin: 10px; background-color: White; padding: 10px}
tr:nth-child(even) {background-color: #E7F5FE;}
</style>
</head>
<body>
"""


endpoints = {
'home': 'Home',
'slug': 'Header',
'slug': 'Header',
'slug': 'Header',
}

base_url = '<your server>'

root_dir = '<your root dir>'

sessions = list()

valid_tokens = ['<your UUID tokens>']


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


def file_timestamp_str(filepath):
    modified = os.path.getmtime(filepath)
    time = str(datetime.fromtimestamp(modified))
    return time


def load_json_content(filepath):
    html = ''
    with open(filepath, 'r') as infile:
        content = json.load(infile)
    for section in content:
        html += '\n<div>\n<h2>%s</h2>\n' % section['Header']
        html += json_to_html_table(section['Data'])
        html += '\n</div>'
    return html


def generate_nav_bar(nav):
    html = ''
    for key in nav.keys():
        html += '<a href="%s/%s">%s</a>&nbsp;&mdash;&nbsp;' % (base_url, key, nav[key])
    return html


def check_session(cookie, ip):
    for session in sessions:
        if session['uuid'] == cookie and session['ip'] == ip:
            return True
    return False


app = flask.Flask('InfrastructureDashboard')


@app.route('/', methods=['get'])
def home():
    return flask.redirect('%s/home' % base_url, code=302)


@app.route('/login', methods=['get', 'post'])
def login():
    valid_session = check_session(flask.request.cookies.get('uuid'), flask.request.remote_addr)
    if valid_session:
        return flask.redirect('%s/home' % base_url, code=302)
    if flask.request.method == 'GET':
        html = base_html
        html += '<h1>Infrastructure Dashboard</h1><form action="%s/login" method="post"><input type="text" placeholder="Token" name="token"><button type="submit">Login</button></form>' % base_url
        return html
    elif flask.request.method == 'POST':
        token = flask.request.form['token']
        if token in valid_tokens:
            cookie = str(uuid.uuid4())
            session = {'uuid': cookie, 'ip': str(flask.request.remote_addr)}
            sessions.append(session)
            response = flask.make_response(flask.redirect(base_url))
            response.set_cookie('uuid', cookie)
            return response
        else:
            html = base_html
            html += '<h1>Infrastructure Dashboard</h1><p>Token not accepted</p><form action="%s/login" method="post"><input type="text" placeholder="Token" name="token"><button type="submit">Login</button></form>' % base_url
            return html


@app.route('/<endpoint>', methods=['get'])
def generic_endpoint(endpoint):
    valid_session = check_session(flask.request.cookies.get('uuid'), flask.request.remote_addr)
    if not valid_session:
        return flask.redirect('%s/login' % base_url, code=302)
    if endpoint == 'favicon.ico':
        return flask.send_from_directory('%s/' % root_dir, 'favicon.ico', mimetype='image/vnd.microsoft.icon')
    if endpoint not in endpoints.keys():
        return 'Endpoint %s not available' % endpoint, 404
    html = base_html
    html += generate_nav_bar(endpoints)
    try:
        html += '\n<h1>%s</h1>\n<p>Last Updated: %s</p>\n<div class="flex-container">' % (endpoints[endpoint], file_timestamp_str('%s/%s.json' % (root_dir, endpoint)))
        html += load_json_content('%s/%s.json' % (root_dir, endpoint))
    except Exception as oops:
        return 'Error loading JSON data: ' + str(oops), 500
    html += '\n</div>\n</body>\n</html>'
    html = html.replace('<title>Dashboard</title>','<title>Dashboard - %s</title>' % endpoints[endpoint])
    return html


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8443, ssl_context='adhoc')
```

# JSON Content

Here's an example of the JSON content. The primary thing is that it's a list of dicts, where the dicts are each section with `Header` and `Data`. `Header` is just a string, the title of the section. 
`Data` is another list of dicts with the actual table data.

```json
[
	{"Data": [
		{},
		{},
		{},
		{}
		],
	"Header": "Section 1 Title"},
	{"Data": [
		{},
		{},
		{},
		{}
		],
	"Header": "Section 2 Title"}
]
```
