---
title: Sign in with Security and Management Open PowerShell
description: How to sign in with Security and Management Open PowerShell as a user, service principal, or access token.
ms.date: 07/20/2020
---

# Sign in with Security and Management Open PowerShell

Security and Management Open PowerShell supports several authentication methods. When writing scripts for automation, the recommended approach is to use an [access token](multi-factor-auth.md) with the necessary permissions. When you restrict sign-in permissions as much as possible for your use case, you help keep your Azure resources secure.

## Sign in interactively

To sign in interactively, use the [Connect-SecMgmtAccount](/powershell/module/secmgmt/connect-secmgmtaccount) cmdlet.

```powershell
Connect-SecMgmtAccount
```

When you run this cmdlet it will open a browser where you will be able to authenticate. If the system you are using to connect does not a browser, then you will be provided a code. To authenticate, copy this code and paste it into <https://microsoft.com/devicelogin> in a browser.

> [!IMPORTANT]
> Username/password credential authorization has been removed in Security and Management Open PowerShell due to the partner security requirements. If you use credential authorization for automation purposes, instead implement the [Secure Application Model](multi-factor-auth.md) so you can authenticate using an access token generated using a refresh token.

## Sign in with a service principal

Service principals are non-interactive Azure accounts. Like other user accounts, their permissions are managed with Azure Active Directory. By granting a service principal only the permissions it needs, your automation scripts stay secure.

To sign in with a service principal, use the `-ServicePrincipal` argument with the `Connect-SecMgmtAccount` cmdlet. You will also need the service principal's application identifier, sign-in credentials, and the tenant identifier associate with the service principal. How you sign in with a service principal will depend on whether it is configured for password-based or certificate-based authentication.

### Certificate-based authentication

Certificate-based authentication requires that Security and Management Open PowerShell can retrieve information from a local certificate store based on a certificate thumbprint.

```powershell
Connect-SecMgmtAccount -CertificateThumbprint '<thumbprint>' -ServicePrincipal -TenantId 'xxxx-xxxx-xxxx-xxxx'
```

In PowerShell 5.1, the certificate store can be managed and inspected with the [PKI](/powershell/module/pkiclient) module. For PowerShell Core 6.x and later, the process is more complicated. The following scripts show you how to import an existing certificate into the certificate store accessible by PowerShell.

#### Import a certificate in PowerShell 5.1

```powershell
# Import a PFX
$credentials = Get-Credential -Message "Provide PFX private key password"
Import-PfxCertificate -FilePath <path to certificate> -Password $credentials.Password -CertStoreLocation cert:\CurrentUser\My
```

#### Import a certificate in PowerShell Core 6.x and later

```powershell
# Import a PFX
$storeName = [System.Security.Cryptography.X509Certificates.StoreName]::My 
$storeLocation = [System.Security.Cryptography.X509Certificates.StoreLocation]::CurrentUser
$store = [System.Security.Cryptography.X509Certificates.X509Store]::new($storeName, $storeLocation)
$certPath = <path to certificate>
$credentials = Get-Credential -Message "Provide PFX private key password"
$flag = [System.Security.Cryptography.X509Certificates.X509KeyStorageFlags]::Exportable
$certificate = [System.Security.Cryptography.X509Certificates.X509Certificate2]::new($certPath, $credentials.Password, $flag)
$store.Open([System.Security.Cryptography.X509Certificates.OpenFlags]::ReadWrite)
$store.Add($Certificate)
$store.Close()
```

## Sign in to another Cloud

Microsoft cloud services offer environments compliant with regional data-handling laws. For accounts in a regional cloud, set the environment when you sign in with the `-Environment` argument. As an example, if your account is in the China cloud:

```powershell
Connect-SecMgmtAccount -Environment AzureChinaCloud
```

The following command gets a list of available environments:

```powershell
Get-PartnerEnvironment | Select-Object Name
```
