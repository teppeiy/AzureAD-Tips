# Azure AD B2B 招待メールの再送スクリプト

## 概要
Azure AD B2B で招待されたユーザーがメールを紛失した等の対応のため、招待メールを再送する必要がある、というケースがあります。  
Azure ポータルのゲストユーザーのプロファイルページより再送が可能ですが、ここへ辿りつくまでに時間がかかる、もっと効率的に再送したい、というご要望があります。その際に役に立つPowerShellスクリプトのサンプルを共有します。

### 招待メールは不要なケースも
招待されたユーザーがすでに Azure AD もしくはマイクロソフトアカウントを保持している場合には、招待メールは不要です。これらのユーザーは、招待されたアプリケーションのURLへ直接アクセスする、もしくは、https://myapps.microsoft.com/[InvitingTenantNamed].onmicrosoft.com へ直接アクセスすることで、招待の承諾が可能です。（例: https://myapps.microsoft.com/contoso.onmicrosoft.com）
詳しくは、[こちら](
https://docs.microsoft.com/ja-jp/azure/active-directory/b2b/redemption-experience#redemption-through-a-direct-link)をご覧ください。  



.ps1ファイルに保存し、ローカル環境等で実行ください。

``` Powershell
#Requires –Version 5

<# 
.SYNOPSIS
	This script is Windows PowerShell sample for Azure AD B2B administrative tasks

.DESCRIPTION
	Version: 1.0.0
	This script is Windows PowerShell sample for Azure AD B2B administrative tasks

.DISCLAIMER
	THIS CODE AND INFORMATION IS PROVIDED "AS IS" WITHOUT WARRANTY OF
	ANY KIND, EITHER EXPRESSED OR IMPLIED, INCLUDING BUT NOT LIMITED TO
	THE IMPLIED WARRANTIES OF MERCHANTABILITY AND/OR FITNESS FOR A
	PARTICULAR PURPOSE.
#>

<# 
 .SYNOPSIS
    Gets Azure AD B2B invitation status

 .DESCRIPTION
    This function returns boolean based on the invitation status.
    It returens True if invitation has not been redeemed, otherwise False.

 .Parameter InvitedUserEmailAddress
    The email address of the user being invited. Required.

 .Example
   $TrueOrFalse = IsPendingAcceptance -InvitedUserEmailAddress "foo@contoso.com"
#>
function IsPendingAcceptance {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory)]
        [string]$InvitedUserEmailAddress
    )
    # Query if there's Pending Acceptance Guest with specified email
    Write-Host "Checking invitation status for" $InvitedUserEmailAddress
    $r = Get-AzureADUser -filter "Creationtype eq 'invitation' and Mail eq '$InvitedUserEmailAddress'" -all $true | Get-MsolUser | Where-Object { $_.AlternativeSecurityIds.Count -eq 0 }

    if($r.count -gt 0) # Found
    {
        Write-Host $InvitedUserEmailAddress " has not accepted the invitation. Returning TRUE"
        return $true
    }
    Write-Host $InvitedUserEmailAddress " has accepted the invitation or has never been invited. Returning FALSE"
    return $false
}

<# 
 .SYNOPSIS
    Resend Azure AD B2B invitation based on the invitation status 

 .DESCRIPTION
    Resend Azure AD B2B invitation based on the invitation status 

 .Parameter InvitedUserEmailAddress
    The email address of the user being invited. Required.

 .Parameter InviteRedirectUrl
    The URL user should be redirected to once the invitation is redeemed. Required.

 .Parameter Force
    If used, it resends invitation email regardless of invitation status

 .Example
    # Send Invitation if user has NOT accepted yet
    ResendInvitation -InvitedUserEmailAddress $InvitedUserEmailAddress -InviteRedirectUrl $InviteRedirectUrl

    # Use Force switch if you want to resend invitation email regardless of invitation status
    ResendInvitation -InvitedUserEmailAddress $InvitedUserEmailAddress -InviteRedirectUrl $InviteRedirectUrl -force
#>
function ResendInvitation {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory)]
        [string]$InvitedUserEmailAddress,
        [Parameter(Mandatory)]
        [string]$InviteRedirectUrl = "https://myapps.microsoft.com/",
        [switch]$Force
    )
    if($force){
        Write-Host -ForegroundColor Green "Resending invitation...."
        New-AzureADMSInvitation -SendInvitationMessage $true -InvitedUserEmailAddress $InvitedUserEmailAddress -InviteRedirectUrl $InviteRedirectUrl
    }
    else{
        if(IsPendingAcceptance -InvitedUserEmailAddress $InvitedUserEmailAddress) { # Found
            Write-Host -ForegroundColor Green "Resending invitation...."
            New-AzureADMSInvitation -SendInvitationMessage $true -InvitedUserEmailAddress $InvitedUserEmailAddress -InviteRedirectUrl $InviteRedirectUrl
        }
        else { # Not Found
            Write-Host -ForegroundColor Red "Do nothing."
        }
    }
}

# Login to Azure AD tenant
Connect-AzureAD

# Login to Azure AD v1 if -Force switch for ResendInvitation function is not used. This is needed for IsPendingAcceptance function.
Connect-MsolService


# Initialize parameters accordingly
$InviteRedirectUrl = "https://myapps.microsoft.com"

# Ask for email address
$InvitedUserEmailAddress = Read-host "Enter email address you want to resend invitation"

# Send Invitation if user has NOT accepted yet
ResendInvitation -InvitedUserEmailAddress $InvitedUserEmailAddress -InviteRedirectUrl $InviteRedirectUrl

# Use Force switch if you want to resend invitation email regardless of invitation status
#ResendInvitation -InvitedUserEmailAddress $InvitedUserEmailAddress -InviteRedirectUrl $InviteRedirectUrl -force
```
