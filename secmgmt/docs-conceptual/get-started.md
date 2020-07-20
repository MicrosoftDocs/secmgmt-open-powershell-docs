---
title: Get started with Security and Management Open PowerShell
description: Learn how to get started with Security and Management Open PowerShell.
ms.date: 07/20/2020
---

# Get started with Security and Management Open PowerShell

Security and Management Open PowerShell is designed for administering and managing Partner Center resources from the command line. This article helps you get started with Security and Management Open PowerShell and teaches the core concepts behind it.

## Install

When you are ready to install Security and Management Open PowerShell on your local machine, follow the instructions in [Install the Security and Management Open PowerShell module](install.md).

## Sign in to Partner Center

Sign in interactively with the `Connect-SecMgmtAccount` cmdlet.

```powershell
Connect-SecMgmtAccount
```

If you are in a non-US region, use the `-Environment` parameter to sign in. Get the name of the environment for your region by using
the [Get-SecMgmtEnvironment](/powershell/module/secmgmt/get-secmgmtenvironment) cmdlet. For example, to sign in to Azure China 21Vianet:

```powershell
Connect-SecMgmtAccount -Environment AzureChinaCloud
```

When you run this cmdlet it will open a browser where you will be able to authenticate. If the system you are using to connect does not a browser, then you will be provided a code. To authenticate, copy this code and paste it into <https://microsoft.com/devicelogin> in a browser.

Once signed in, use the Security and Management Open PowerShell cmdlets to access and manage resources. To learn more about the sign-in process and authentication methods, see [Sign in with Security and Management Open PowerShell](authenticate.md).

## Find commands

Security and Management Open PowerShell cmdlets follow a standard naming convention for PowerShell, `VERB-NOUN`. The verb describes the action (examples include `New`, `Get`, `Set`, `Remove`) and the noun describes the resource type. Nouns in Security and Management Open PowerShell always start with the prefix `SecMmgtAccount`. For the full list of standard verbs, see [Approved verbs for PowerShell Commands](/powershell/developer/cmdlet/approved-verbs-for-windows-powershell-commands).

You can use the [Get-Command](/powershell/module/microsoft.powershell.core/get-command) cmdlet to help find the nouns and verbs available. As an example, to find all Customer commands that use the `Get` verb you would use the following

```powershell
# List all cmdlets in the SecMgmt module
Get-Command -Module SecMgmt

# List all cmdlets that contain Hybrid
Get-Command -Name '*Hybrid*'

# List all cmdlets that contain Hybrid in the SecMgmt module
Get-Command -Module SecMgmt -Name '*Hybrid*'
```

## Next steps

* [Sign in with Security and Management Open PowerShell](authenticate.md)
