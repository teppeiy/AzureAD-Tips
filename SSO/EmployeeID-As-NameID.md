# EmployeeIDをユーザー識別子として利用したい
2017/10現在、EmployeeIDはAzureポータルにてクレームマッピングポリシーを設定することができないため、PowerShellを利用して設定する必要があります。

https://docs.microsoft.com/en-us/azure/active-directory/active-directory-claims-mapping

```Powershell
# Create Policy
# SAMLトークンのNameidentifierクレームに、employeeId属性を利用するポリシーの作成
$newPolicy = New-AzureADPolicy -Definition @('{"ClaimsMappingPolicy":{"Version":1,"IncludeBasicClaimSet":"true", "ClaimsSchema": [{"Source":"user","ID":"employeeid","SamlClaimType":"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier","JwtClaimType":"nameidentifier"}]}}') -DisplayName "EmployeeIdAsNameId" -Type "ClaimsMappingPolicy" -IsOrganizationDefault $false

# Getting Service Principal
# アプリ（サービスプリンシパル）オブジェクトの取得
$targetSPName = "Your App DisplayName"
$targetSP = Get-AzureADServicePrincipal -SearchString $targetSPName

# 対象のサービスプリンシパルオブジェクトを表示
Write-Host "Target Service Principal"
Write-host $targetSP

# ポリシーとサービスプリンシパルのオブジェクトIDを取得
$targetPolicyId = $newPolicy.Id
$targetSpObjectId = $targetSP.ObjectId

# Assigning Policy to Service Principal
# サービスプリンシパルにクレームマッピングポリシーをリンク
Add-AzureADServicePrincipalPolicy -Id $targetSpObjectId -RefObjectId $targetPolicyId

# Confirm Policy mapping to Service Principal
# リンクされたポリシーを確認
Get-AzureADServicePrincipalPolicy -Id $targetSpObjectId

# Remove Policy from Service Principal
# ポリシーの削除
Remove-AzureADServicePrincipalPolicy -Id $targetSpObjectId -PolicyId $targetPolicyId
```

### クレームマッピングポリシーの例
Nameidentifer = Join(user.employeeId + '@' + domain)
```Powershell
New-AzureADPolicy -Definition @('{"ClaimsMappingPolicy":{"Version":1,"ClaimsSchema":[{"Source":"User","ID":"EmployeeID","DisplayName":"EmployeeID","SamlClaimType":"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/employeeid","JwtClaimType":"employeeid"},{"ID":"NameId","Source":"Transformation","SamlClaimType":"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier","TransformationId":"AddSuffixToEmployeeId"}],"ClaimsTransformations":[{"ID":"AddSuffixToEmployeeId","TransformationMethod":"Join","InputClaims":[{"ClaimTypeReferenceId":"EmployeeID","TransformationClaimType":"string1"}],"InputParameters":[{"ID":"string2","Value":"contoso.com"},{"ID":"separator","Value":"@"}],"OutputClaims":[{"ClaimTypeReferenceId":"NameId","TransformationClaimType":"outputClaim"}]}]}}') -DisplayName "Gid+Domain" -Type "ClaimsMappingPolicy" -IsOrganizationDefault $false
```