---
layout: default
title: "Installing PowerShell modules behind corporate proxy"
date: 2021-03-12
description: WARNING! Unable to find module repositories! No match was found for the specified search criteria and module name 'PowerShellGet'!
categories: [PowerShell]
---

# Symptoms

You see this kinda thing:

```powershell
PS C:\WINDOWS\system32> Register-PSRepository -Default -verbose
VERBOSE: Performing the operation "Register Module Repository." on target "Modul
e Repository 'PSGallery' () in provider 'PowerShellGet'.".

PS C:\WINDOWS\system32> Get-PSRepository
WARNING: Unable to find module repositories.
```

NOTE: The above commands should look more like this:

```powershell
PS C:\WINDOWS\system32> Register-PSRepository -Default -verbose
VERBOSE: Performing the operation "Register Module Repository." on target "Modul
e Repository 'PSGallery' () in provider 'PowerShellGet'.".
VERBOSE: Repository details, Name = 'PSGallery', Location = 'https://www.powersh
ellgallery.com/api/v2'; IsTrusted = 'False'; IsRegistered = 'True'.

PS C:\WINDOWS\system32> Get-PSRepository

Name                      InstallationPolicy   SourceLocation                                                                
----                      ------------------   --------------                                                                
PSGallery                 Untrusted            https://www.powershellgallery.com/api/v2                                      
```

Other symptoms:

```powershell
PS C:\WINDOWS\system32> Install-Module -Name PowerShellGet -Scope AllUsers -Confirm:$false -Force -AllowClobber
PackageManagement\Install-Package : No match was found for the specified 
search criteria and module name 'PowerShellGet'. Try Get-PSRepository to see 
all available registered module repositories.
At C:\Program 
Files\WindowsPowerShell\Modules\PowerShellGet\1.0.0.1\PSModule.psm1:1809 
char:21
+ ...          $null = PackageManagement\Install-Package @PSBoundParameters
+                      ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (Microsoft.Power....InstallPacka 
   ge:InstallPackage) [Install-Package], Exception
    + FullyQualifiedErrorId : NoMatchFoundForCriteria,Microsoft.PowerShell.Pac 
   kageManagement.Cmdlets.InstallPackage
```

# Solution

## Setup Proxy

First you'll need to tell your PowerShell session to use your proxy. You might also need to change the TLS protocol that's accepted. Basically, what's happening above is just a bad error message. It's not verbose enough to say "HTTPS failure" or anything like that.

```powershell
$proxy = '<YOUR CORPORATE PROXY HERE>'  # update this
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
[system.net.webrequest]::defaultwebproxy = new-object system.net.webproxy($proxy)
[system.net.webrequest]::defaultwebproxy.credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials
[system.net.webrequest]::defaultwebproxy.BypassProxyOnLocal = $true
```

## Update Package Providers

This is similar-ish to `sudo apt-get update`. The first line gives you the package provider itself, the second registers PSGallery and the third installs PowerShellGet, which is basically an installer that a lot of packages use.

```powershell
Install-PackageProvider -Name nuget -Scope AllUsers -Confirm:$false -Force -MinimumVersion 2.8.5.201
Register-PSRepository -Default -verbose
Install-Module -Name PowerShellGet -Scope AllUsers -Confirm:$false -Force -AllowClobber -MinimumVersion 2.2.4 -SkipPublisherCheck
```

## Close PowerShell

I have no idea why this is required. But trust me, you'll continue to get weird errors on some of your installations, such as the following:

```powershell
PS C:\Windows\system32> Install-Module -Name Cisco.UCS.Core 
WARNING: The specified module 'Cisco.UCS.Core' with PowerShellGetFormatVersion '2.0' is not supported by the current version of PowerShellGet. Get the latest version of the PowerShellGet module to install this module, 'Cisco.UCS.Core'.
```

Once again, this is a sign of bad error messages. But what else is new from Microsoft? This has been the case for 20+ years. 

## Install other modules

Now you should be able to install whatever else you need. Note, you may need to always run the first block before subsequent updates and such. But first you need to rerun the security stuff at the beginning.

```powershell
Install-Module -Name VMware.PowerCLI -Scope AllUsers -Confirm:$false -Force -AllowClobber
Install-Module -Name Cisco.UCS.Core -Scope AllUsers -Confirm:$false -Force -AllowClobber -AcceptLicense
Install-Module -Name Cisco.UCSManager -Scope AllUsers -Confirm:$false -Force -AllowClobber -AcceptLicense
Install-Module -Name Az -Scope AllUsers -Confirm:$false -Force -AllowClobber
```

# Bringing it all together

Here's how it looks if you want to run it as a single script. I like to do this and save it in a github repo so I can quickly install stuff on servers/laptops without remembering all this nonsense. 

Except you can't run it all together because for some reason you must completely close PowerShell before installing more modules. >:|

## Script 1

```powershell
$proxy = '<YOUR CORPORATE PROXY HERE>'  # update this
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
[system.net.webrequest]::defaultwebproxy = new-object system.net.webproxy($proxy)
[system.net.webrequest]::defaultwebproxy.credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials
[system.net.webrequest]::defaultwebproxy.BypassProxyOnLocal = $true

Install-PackageProvider -Name nuget -Scope AllUsers -Confirm:$false -Force -MinimumVersion 2.8.5.201
Register-PSRepository -Default -verbose
Install-Module -Name PowerShellGet -Scope AllUsers -Confirm:$false -Force -AllowClobber -MinimumVersion 2.2.4 -SkipPublisherCheck

exit
```

## Script 2

```powershell
$proxy = '<YOUR CORPORATE PROXY HERE>'  # update this
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
[system.net.webrequest]::defaultwebproxy = new-object system.net.webproxy($proxy)
[system.net.webrequest]::defaultwebproxy.credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials
[system.net.webrequest]::defaultwebproxy.BypassProxyOnLocal = $true

Install-Module -Name VMware.PowerCLI -Scope AllUsers -Confirm:$false -Force -AllowClobber
Install-Module -Name Cisco.UCS.Core -Scope AllUsers -Confirm:$false -Force -AllowClobber -AcceptLicense
Install-Module -Name Cisco.UCSManager -Scope AllUsers -Confirm:$false -Force -AllowClobber -AcceptLicense
Install-Module -Name Az -Scope AllUsers -Confirm:$false -Force -AllowClobber
```
