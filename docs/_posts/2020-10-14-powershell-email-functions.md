---
layout: default
title: "PowerShell Email Functions"
date: 2020-10-14
description: Since I'm a monitoring addict, I use email reports a lot...
categories: [PowerShell, Email, Monitoring, KB]
---

These are some functions I keep in a PowerShell module on my Rundeck server for easy reuse. I can import the module at the beginning of jobs like so: 

```powershell
Import-Module -Force -DisableNameChecking "C:\<modules_directory>\EmailFunctions.psm1"
```

# Send HTML Email

For the most part, you will want some formatting, which means you need HTML support. In my experience, most organizations have internal SMTP relays that will relay to local (domain) addresses without authentication. This is pretty typical as many servers, applications, and hardware support email notifications but many do not support SMTP authentication. 

```powershell
function Send-EmailHtml
    {
    param($to, $from, $subject, $html, $smtp)
    $message = New-Object System.Net.Mail.MailMessage $from, $to
    $message.Subject = $subject
    $message.IsBodyHTML = $true
    $message.body = $html
    $smtp = New-Object Net.Mail.SmtpClient($smtp)  # FQDN of your SMTP server or relay
    $smtp.Send($message)
    }
```

# Make a pretty HTML table

PowerShell already has a default function for converting data objects to HTML tables but it's ugly. 

```powershell
function Make-HtmlTable
    {
    param($data)
    $t_header = @"
<style>
TABLE {
    border-width: 1px; 
    border-style: solid; 
    border-color: lightgray; 
    border-collapse: collapse; 
    font-family: "Helvetica", arial, sans-serif;
    font-size: 12px;
    white-space: nowrap;
    }
TH {border-width: 1px; padding: 4px; border-style: solid; border-color: lightgray; background-color: lightskyblue; white-space: nowrap;}
TD {border-width: 1px; padding: 4px; border-style: solid; border-color: lightgray; white-space: nowrap;}
</style>
"@
    $table = $data | ConvertTo-Html -Head $t_header
    return $table
    }
```

# Send email with attachment

I have, on occasion, been asked to schedule reports for other people. Sometimes they don't want it in a pretty HTML table, they want it as an Excel doc or something. For that, I recommend [PowerShell Galleries ImportExcel](https://www.powershellgallery.com/packages/ImportExcel/7.1.1). 

```powershell
function Send-EmailAttachment
    { 
    param($to, $from, $subject, $body, $attachment, $smtp)
    # attachment must be in the form of full file path to attachment
    $message = New-Object System.Net.Mail.MailMessage $from, $to
    $message.Subject = $subject
    $message.IsBodyHTML = $true
    $message.Body = $body
    $file = new-object Net.Mail.Attachment($attachment) 
    $message.Attachments.Add($file) 
    $smtp = New-Object Net.Mail.SmtpClient($smtp)
    $smtp.Send($message)
    }
```
