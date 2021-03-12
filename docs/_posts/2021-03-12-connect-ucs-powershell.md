---
layout: default
title: "Connect to UCSM with PowerShell and specify your domain"
date: 2021-03-12
description: UCSM requires a particular syntax to connect
categories: [PowerShell, KB, UCS]
---

# Symptom

You are unable to connect to a UCS instance by passing a PowerShell credential object.

# Cause

You must specify the authentication scope in the username of the PowerShell credential object.

# Solution

Prepend your PowerShell credential username with the following:

- `ucs-local\<username>` for local auth
- `ucs-<DOMAIN>\<username>` for LDAP/AD auth

Examples:

- `ucs-local\admin` for local admin
- `ucs-RATPACK\frank.sinatra` for AD auth

If you use `Get-Credential` in a script, you can remind yourself by using the prompt:

```powershell
$creds = Get-Credential -Message "UCS Auth" -UserName "ucs-<DOMAIN>\"
Connect-Ucs -Name ucsm.contoso.com -Credential $creds
```
