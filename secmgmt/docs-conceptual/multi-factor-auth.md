---
title: Automating tasks with PowerShell
description: How to automate tasks in PowerShell when Multi-Factor Authentication is enforced.
ms.date: 07/20/2020
---

# Multi-Factor Authentication

Multi-Factor Authentication (MFA) helps safeguard access to data and applications while maintaining simplicity for users. It provides additional security by requiring a second form of authentication and delivers strong authentication through a range of easy to use authentication methods. Users may or may not be challenged for Multi-Factor Authentication based on configuration decisions that an administrator makes. 

## Secure Application Model

The requirement for Multi-Factor Authentication can complicate any automation that you have developed because a second form of authentication must be provided when authenticating. To content with this requirement, the Secure Application Model was developed to provide guidance on how the appropriate authentication can be performed in non-interactive scenarios. This model is comprised of two distinct steps

| Step | Description |
| ---- | ----------- |
| Consent  | This where you will authenticate interactively using the [authorization code flow](/azure/active-directory/develop/v2-oauth2-auth-code-flow) or [device code flow](/azure/active-directory/develop/v2-oauth2-device-code). The response from Azure Active Directory will contain an access token and a refresh token. The refresh token value should be stored somewhere secure, such as [Azure Key Vault](/azure/key-vault/key-vault-whatis). This value will be used by your application, or script, instead of user credential when authenticating.  |
| Exchange | Using the securely stored refresh token, generated through the consent step, you will request a new access token from Azure Active Directory. See [refresh the access token](/azure/active-directory/develop/v2-oauth2-auth-code-flow#refresh-the-access-token) for more information regarding the refresh token value. |

> [!IMPORTANT]
> By default, the lifetime of a refresh token is 90 days. So, it is important that you have a process for updating the refresh token prior to the expiration. If it does expire, you will receive an error similar to the following when attempting to exchange it for an access token *The refresh token has expired due to inactivity. The token was issued on 2019-01-02T09:19:53.5422744Z and was inactive for 90.00:00:00*.

### Consent

The consent step can be performed through several different methods. When using PowerShell it is recommended to use the [New-SecMgmtAccessToken](/powershell/module/secmgmt/new-secmgmtaccesstoken) cmdlet. The following is an example of how you can request a new access token for use with the Microsoft Graph or this PowerShell module.

```powershell
$credential = Get-Credential
New-SecMgmtAccessToken -ApplicationId 'xxxx-xxxx-xxxx-xxxx' -Scopes 'https://graph.microsoft.com/.default' -ServicePrincipal -Credential $credential -Tenant 'yyyy-yyyy-yyyy-yyyy' -UseAuthorizationCode
```

> [!IMPORTANT]
> When using the `UseAuthorizationCode` parameter you will be prompted to authentication interactively using the authorization code flow. The redirect URI value will generated dynamically. This generation process will attempt to find a port between 8400 and 8999 that is not in use. Once an available port has been found, the redirect URL value will be constructed (e.g. <http://localhost:8400>). So, it is important that you have configured the redirect URI value for your Azure Active Directory application accordingly.

The first command gets the service principal credentials (application identifier and secret), and then stores them in the `$credential` variable. The second command will generate a new access token using the service principal credentials stored in the `$credential` variable and the authorization code flow. The output from this command will contain several values, including a refresh token. That value should be stored somewhere secure such as [Azure Key Vault](/azure/key-vault/key-vault-whatis) because it will be used instead of user credentials in future operations.

### Exchange

The exchange step can be performed through a number of different methods. When using PowerShell it is recommended to use the [New-SecMgmtAccessToken](/powershell/module/secmgmt/new-secmgmtaccesstoken) cmdlet. The following is an example of how to exchange a refresh token for an access token.

```powershell
$credential = Get-Credential
$refreshToken = '<refreshToken>'

New-SecMgmtAccessToken -ApplicationId 'xxxx-xxxx-xxxx-xxxx' -Credential $credential -RefreshToken $refreshToken -Scopes 'https://graph.microsoft.com/.default' -ServicePrincipal -Tenant 'yyyy-yyyy-yyyy-yyyy'
```

The first command gets the service principal credentials (application identifier and secret), and then stores them in the `$credential` variable. The third command will generate a new access token using the service principal credentials stored in the `$credential` variable and the refresh token stored in the `$refreshToken` variable for authentication.

## Samples

The following sections demonstrate how to use the [New-SecMgmtAccessToken](/powershell/module/secmgmt/new-secmgmtaccesstoken) cmdlet to request access tokens and connect to other commonly used PowerShell modules.

### Azure

#### Azure PowerShell

```powershell
$credential = Get-Credential
$refreshToken = '<RefreshToken>'

$azureToken = New-SecMgmtAccessToken -ApplicationId 'xxxx-xxxx-xxxx-xxxx' -Credential $credential -RefreshToken $refreshToken -Scopes 'https://management.azure.com//user_impersonation' -ServicePrincipal -Tenant 'yyyy-yyyy-yyyy-yyyy'
$graphToken = New-SecMgmtAccessToken -ApplicationId 'xxxx-xxxx-xxxx-xxxx' -Credential $credential -RefreshToken $refreshToken -Scopes 'https://graph.windows.net/.default' -ServicePrincipal -Tenant 'yyyy-yyyy-yyyy-yyyy'

# Az Module
Connect-AzAccount -AccessToken $token.AccessToken -AccountId 'azureuser@contoso.com' -GraphAccessToken $graphToken.AccessToken -TenantId 'xxxx-xxxx-xxxx-xxxx'
```

> [!NOTE]
> When connecting to an environment where you have admin on behalf of privileges, you will need to specify the tenant identifier for the target environment through the `Tenant` parameter. With respect to the Cloud Solution Provider program this means you will specify the tenant identifier of the customer's Azure Active Directory tenant using the `Tenant` parameter.

### Microsoft 365

#### Azure Active Directory

```powershell
$credential = Get-Credential
$refreshToken = '<RefreshToken>'

$aadGraphToken = New-SecMgmtAccessToken -ApplicationId 'xxxx-xxxx-xxxx-xxxx' -Credential $credential -RefreshToken $refreshToken -Scopes 'https://graph.windows.net/.default' -ServicePrincipal -Tenant 'yyyy-yyyy-yyyy-yyyy'
$graphToken = New-SecMgmtAccessToken -ApplicationId 'xxxx-xxxx-xxxx-xxxx' -Credential $credential -RefreshToken $refreshToken -Scopes 'https://graph.microsoft.com/.default' -ServicePrincipal -Tenant 'yyyy-yyyy-yyyy-yyyy'

Connect-AzureAD -AadAccessToken $aadGraphToken.AccessToken -AccountId 'azureuser@contoso.com' -MsAccessToken $graphToken.AccessToken
```

> [!NOTE]
> When connecting to an environment where you have admin on behalf of privileges, you will need to specify the tenant identifier for the target environment through the `Tenant` parameter. With respect to the Cloud Solution Provider program this means you will specify the tenant identifier of the customer's Azure Active Directory tenant using the `Tenant` parameter.

#### Exchange Online PowerShell

When generating the initial refresh token, you will need to use following

```powershell
$token = New-SecMgmtAccessToken -Module ExchangeOnline
``` 

> [!IMPORTANT]
> After invoking the commands above, you will find the refresh token value is available through `$token.RefreshToken`. This value should be stored in a secure repository such as [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) to ensure that is appropriately secure because it will be used instead of credentials.

Use the following to generate a new access token using the refresh, and then create the session where you will connect to Exchange Online PowerShell

```powershell
$customerId = '<CustomerId>'
$customerDomainName = '<CustomerDomainName>'
$refreshToken = '<RefreshTokenValue>'
$upn = '<UPN-used-to-generate-the-refresh-token>'

$token = New-SecMgmtAccessToken -RefreshToken $token.RefreshToken -Scopes 'https://outlook.office365.com/.default' -Tenant $customerId -ApplicationId 'a0c73c16-a7e3-4564-9a95-2bdf47383716'

$tokenValue = ConvertTo-SecureString "Bearer $($token.AccessToken)" -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential($upn, $tokenValue)

$session = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri "https://outlook.office365.com/powershell-liveid?DelegatedOrg=$($customerDomainName)&BasicAuthToOAuthConversion=true" -Credential $credential -Authentication Basic -AllowRedirection

Import-PSSession $session
```

> [!IMPORTANT]
> The application identifier *a0c73c16-a7e3-4564-9a95-2bdf47383716* is for the Exchange Online PowerShell Azure Active Direcotry application. When requesting an access, or refresh, token for use with Exchange Online PowerShell you will need to use this value. 

#### MS Online

```powershell
$credential = Get-Credential
$refreshToken = '<RefreshToken>'

$aadGraphToken = New-SecMgmtAccessToken -ApplicationId 'xxxx-xxxx-xxxx-xxxx' -Credential $credential -RefreshToken $refreshToken -Scopes 'https://graph.windows.net/.default' -ServicePrincipal -Tenant 'yyyy-yyyy-yyyy-yyyy'
$graphToken = New-SecMgmtAccessToken -ApplicationId 'xxxx-xxxx-xxxx-xxxx' -Credential $credential -RefreshToken $refreshToken -Scopes 'https://graph.microsoft.com/.default' -ServicePrincipal -Tenant 'yyyy-yyyy-yyyy-yyyy'

Connect-MsolService -AdGraphAccessToken $aadGraphToken.AccessToken -MsGraphAccessToken $graphToken.AccessToken
```
