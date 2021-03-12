---
layout: default
title: "Rundeck ACL hell"
date: 2021-02-05
description: Rundeck CE ACLs are byzantine
categories: [Rundeck, Automation, KB]
---

# Rundeck ACL YAML files

Even looking at the templates is not very helpful. This is one place where Rundeck's documentation still leaves a bit to be desired.

## Projects vs System

The first thing to know is that Rundeck ACLs have two primary scopes: the projects and the Rundeck system itself. These are demarced by `context: application: 'rundeck'` and `context: project: '.*'`

## Default Admin ACL

I have found that the default admin ACL is the most reliable. Fiddling with other stuff just breaks things. This is probably a Layer 8 issue, though. Because of this, I have learned to base all my other ACLs from this template. 

```yaml
description: Admin access to PROJECTS
context:
  project: '.*' 
for:
  resource:
    - allow: '*' 
  adhoc:
    - allow: '*' 
  job: 
    - allow: '*' 
  node:
    - allow: '*' 
by:
  username:
    - user.name  # case sensitive
  group:
    - admin
    - group.name  # case sensitive

---

description: Admin access to RUNDECK system
context:
  application: 'rundeck'
for:
  resource:
    - allow: '*' 
  project:
    - allow: '*' 
  project_acl:
    - allow: '*' 
  storage:
    - allow: '*' 
by:
  username:
    - user.name  # case sensitive
  group:
    - admin
    - group.name  # case sensitive
```

## Global Read/Run ACL

Obviously, you don't want to give admin privileges to everyone so you'll want at least one more policy to allow some folks to run jobs but not make any other changes. Here's an example that I got working.

```yaml
description: Read/Run access to all PROJECTS
context:
  project: '.*' 
for:
  resource:
    - allow: [read,run,refresh]
  adhoc:
    - allow: [read,run,refresh]
  job: 
    - allow: [read,run,refresh]
  node:
    - allow: [read,run,refresh]
by:
  username:
    - user.name  # case sensitive
  group:
    - group.name  # case sensitive

---

description: Read access to RUNDECK system
context:
  application: 'rundeck'
for:
  resource:
    - allow: read
  project:
    - allow: read
  project_acl:
    - allow: read
  storage:
    - allow: read
by:
  username:
    - user.name  # case sensitive
  group:
    - group.name  # case sensitive
```    

## Read/Run access to specific project

Let's say you want to give one person or team access to a specific project to run jobs. This is pretty common. In one example, I needed to give DBAs access to take storage snapshots. So I created a self service project for DBAs and gave them exclusive access.

```yaml
description: Read/Run access to self service
context:
  project: 'DBA_Self_Service'  # case sensitive, put your project here
for:
  resource:
    - allow: [read,run,refresh]
  adhoc:
    - allow: [read,run,refresh]
  job: 
    - allow: [read,run,refresh]
  node:
    - allow: [read,run,refresh]
by:
  username:
    - user.name  # case sensitive
  group:
    - group.name  # case sensitive

---

description: Read access to RUNDECK system
context:
  application: 'rundeck'
for:
  resource:
    - allow: read
  project:
    - allow: read
  project_acl:
    - allow: read
  storage:
    - allow: read
by:
  username:
    - user.name  # case sensitive
  group:
    - group.name  # case sensitive
```

# Caveat

I'm not an expert. One flaw with this scheme is that anyone who logs in will see all projects listed on the left-hand column. Even so, they will be empty. I tried restricting it further but that just broke it. I don't know if I did it wrong or if it's a bug. As I said, I've been fiddling with ACLs and found this is the most reliable way so far. If I figure out a better scheme, I'll post an update. 

## LDAP is case sensitive

Active Directory and Windows might not be case sensitive, but LDAP authentication is! I just found this out the hard way... 

## Global read policy

You could probably break out the global RUNDECK system read ACL into its own policy instead of recreating it. I've seen that before. Something like "Domain Users" is granted global read. However, this is not necessarily best practice because you don't want Joe Shmoe to see the results of every other rundeck job. 




