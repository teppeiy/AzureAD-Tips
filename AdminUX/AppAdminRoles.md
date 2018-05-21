# アプリケーション管理者権限の委譲

<!--

サンプルスクリプト
``` Powershell
Connect-AzureAD

# Change $roleName and $userPrincipalName accordingly
$roleName = "Application Developer"
$userPrincipalName = "USER@CONTOSO.COM"
 
$role = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq $roleName}
 
if ($role -eq $null) {
    # Instantiate an instance of the role template
    $roleTemplate = Get-AzureADDirectoryRoleTemplate | Where-Object {$_.displayName -eq $roleName}
    Enable-AzureADDirectoryRole -RoleTemplateId $roleTemplate.ObjectId
 
    $role = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq $roleName}
}
 
$roleMember = Get-AzureADUser -ObjectId $userPrincipalName
Add-AzureADDirectoryRoleMember -ObjectId $role.ObjectId -RefObjectId $roleMember.ObjectId
Get-AzureADDirectoryRoleMember -ObjectId $role.ObjectId | Get-AzureADUser
```

-->