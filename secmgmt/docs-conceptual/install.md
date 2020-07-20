---
title: Install Security and Management Open PowerShell with PowerShellGet
description: How to install Security and Management Open PowerShell with PowerShellGet
ms.date: 07/20/2020
---

# Security and Management Open PowerShell

Security and Management Open PowerShell is commonly used by partners to manage their Partner Center resources. Using this module you can perform tasks such as customer and subscription life cycle management, confirm customer acceptance of the Microsoft Customer Agreement, and purchase reserved instances.

## Requirements

Security and Management Open PowerShell works with PowerShell 5.1 or higher on Windows, or PowerShell Core 6.x and later on all platforms. If you are not sure if you have PowerShell, or using macOS or Linux, [install the latest version of PowerShell Core](/powershell/scripting/install/installing-powershell#powershell-core).

To check your PowerShell version, run the following command:

```powershell
$PSVersionTable.PSVersion
```

To run Security and Management Open PowerShell using PowerShell 5.1 on Windows:

1. Update to [Windows PowerShell 5.1](/powershell/scripting/install/installing-windows-powershell#upgrading-existing-windows-powershell) if needed. If you are on Windows 10, you already have PowerShell 5.1 installed.
2. Install [.NET Framework 4.7.2 or later](/dotnet/framework/install).

There are no additional requirements for Security and Management Open PowerShell when using PowerShell Core.

## Install the Security and Management Open PowerShell module

The recommended install method is to only install for the active user:

```powershell
Install-Module -Name SecMgmt -AllowClobber -Scope CurrentUser
```

If you want to install for all users on a system, this requires administrator privileges. From an elevated PowerShell session either run as administrator or with the `sudo` command on macOS or Linux:

```powershell
Install-Module -Name SecMgmt -AllowClobber -Scope AllUsers
```

By default, the PowerShell gallery is not configured as a trusted repository for PowerShellGet. The first time you use the PSGallery you see the following prompt:

```output
Untrusted repository

You are installing the modules from an untrusted repository. If you trust this repository, change
its InstallationPolicy value by running the Set-PSRepository cmdlet.

Are you sure you want to install the modules from 'PSGallery'?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"):
```

Answer `Yes` or `Yes to All` to continue with the installation.

## Troubleshooting

Here are some common problems seen when installing the Security and Management Open PowerShell module. If you experience a problem not listed here,
please [file an issue on GitHub](https://github.com/microsoft/secmgmt-open-powershell/issues).

### Proxy blocks connection

If you get errors from `Install-Module` that indicate the PowerShell Gallery is unreachable, you may be behind a proxy. Different operating systems will have different requirements for configuring a system-wide proxy, which are not covered in detail here. Contact your system administrator for your proxy settings and how to configure them for your OS.

PowerShell itself may not be configured to use this proxy automatically. With PowerShell 5.1 and later, configure the proxy to use for a PowerShell session with the following command:

```powershell
(New-Object System.Net.WebClient).Proxy.Credentials = `
  [System.Net.CredentialCache]::DefaultNetworkCredentials
```

If your operating system credentials are configured correctly, this will route PowerShell requests through the proxy.
In order to have this setting persist between sessions, add the command to a
[PowerShell profile](/powershell/module/microsoft.powershell.core/about/about_profiles).

In order to install the package, your proxy needs to allow HTTPS connections to the following address:

* `https://www.powershellgallery.com`

## Sign in

To start working sign in with your Azure Active Directory credentials.

```powershell
Connect-SecMgmtAccount
```

## Provide feedback

If you find a bug in Security and Management Open PowerShell or have feedback, [file an issue on GitHub](https://github.com/microsoft/secmgmt-open-powershell/issues).