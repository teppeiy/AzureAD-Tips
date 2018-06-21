# ユーザーID (UPN) の入力を省きたい

Docsへ同様な記事を公開しました  
https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-auto-acceleration-using-hrd


## シナリオ
ADFSとフェデレーション環境において、SP Initiated SSOフローの際、Azure ADのログイン画面にユーザーID (UPN)を入力する必要があります。これはAzure ADがユーザーのテナントを特定するために必要なプロセスで、これをホームレルムディスカバリー (HRD) と呼びます。  
ユーザーエクスペリエンスを向上させるため、このステップ (ユーザーがログインIDを入力する手間) を省く方法 (HRDアクセラレーション) が3つあります。 
### 1. ユーザーのアクセスURLを利用する

Azureポータルの、[エンタープライズアプリケーションブレード]より対象アプリケーションを選択、プロパティメニューの「ユーザーアクセスURL」にユーザーのドメイン情報を付加し、そのリンクをユーザーに利用してもらいます。

方法1：myapps.microsoft.com のスラッシュ(/)の後に対象ユーザーのUPNドメインを付加します。
```URL
https://myapps.microsoft.com/contoso.com/signin/AppName/2a354c93-d50f-47ed-b303-dc65f5af7a38
```
方法2：ユーザーアクセスURLのパラメーターとして、whr=[対象ユーザーのUPNドメイン]を付加します。
```URL
https://myapps.microsoft.com/signin/AppName/2a354c93-d50f-47ed-b303-dc65f5af7a38?whr=contoso.com
```


### 2. アプリケーションで対応
アプリケーションがIdentity ProviderであるAzure ADへ適切なパラメータを引渡すことで、HRDアクセラレーションが可能です。  
Office 365やAzure ADアクセスパネルは、HRDアクセラレーションに対応しています。以下、contoso.com というUPNドメインが登録されている Azure AD テナントに対するHRDアクセラレーションの例です。

Outlook Web App
```URL
https://outlook.office.com/contoso.com
```

Azure AD アクセスパネル
```URL
https://myapps.microsoft.com/contoso.com
```
アプリケーションが対応している、もしくは改変で対応が可能であれば、同様にURLパラメータ等でHRDアクセラレーションが可能です。詳しくは、[こちらのブログ](https://blogs.msdn.microsoft.com/tsmatsuz/2015/04/20/azure-ad-custom-branding-login-ui-home-realm-discovery-domain-hint/)をご参考ください。  
  

### 3. SPに指定するIdPのURLにレルム情報を指定する (SAML-P)
1に似ているのですが、SAMLに限ってSPの設定で指定するIdP（Azure ADのSAMLエンドポイント）にWHRパラメータ(whr=contoso.com)を追記することで、対象のAzure ADテナントへリダイレクトされます。  
```XML
https://login.microsoftonline.com/23fbfc88-a7d3-49b0-9e68-3d6922aa9ac6/saml2?whr=contoso.com
```

### 4. Azure ADのServicePrincipalにHRDアクセラレーションポリシーを適用する
[こちら](https://support.office.com/ja-jp/article/%E8%87%AA%E5%8B%95%E3%82%A2%E3%82%AF%E3%82%BB%E3%83%A9%E3%83%AC%E3%83%BC%E3%82%BF-%E3%83%9D%E3%83%AA%E3%82%B7%E3%83%BC%E3%82%92%E4%BD%BF%E7%94%A8%E3%81%97%E3%81%A6-Yammer-%E3%81%AB%E5%AF%BE%E3%81%99%E3%82%8B-Office-365-%E3%82%B5%E3%82%A4%E3%83%B3%E3%82%A4%E3%83%B3%E3%82%92%E6%94%B9%E5%96%84%E3%81%99%E3%82%8B-4d0e5067-992c-4cd6-bad5-b4ac0d52f596?ui=ja-JP&rs=ja-JP&ad=JP)はYammerでの例ですが、汎用的に利用可能です。[サンプルスクリプト](#HRDアクセラレーションポリシー適用のPowerShellサンプル)も参考ください。

#### 2. 3. の注意点
ADFS等とフェデレーションしているドメインに対してHRDアクセラレーションをする場合、アプリのサインオンURLに移動後、問答無用でADFSにリダイレクトされます。その場合、対象アプリにクラウドIDの利用者や、B2Bユーザー、また、別なADFSでログインするユーザーが居る場合にはこの手法は利用することが不可です。


### HRDアクセラレーションポリシー適用のPowerShellサンプル
```PowerShell
# 必要に応じて
# Azure AD Previewモジュールのインストール・インポート（PowerShellを管理者権限で実行）
Install-Module -Name AzureADPreview -Force
Import-Module -Name AzureADPreview

# Login
Connect-AzureAD

########################################################################
# 設定

# ADFSへフェデレーションされているドメイン
$federatedDomain = "contoso.com"

# Azure ADのアプリケーション（Service Principal）の表示名（DisplayName）
$targetSPName = "Concur"

########################################################################

# Azure ADへ接続
Connect-AzureAD

# テナント上のポリシーの確認
Get-AzureAdPolicy | Where-Object {$_.Type -eq "HomeRealmDiscoveryPolicy"}

# ポリシーの作成
$newPolicy = New-AzureADPolicy -Definition @("{`"HomeRealmDiscoveryPolicy`":{`"AccelerateToFederatedDomain`":true,`"PreferredDomain`":`"$federatedDomain`"}}") -DisplayName BasicAutoAccelerationPolicy -Type HomeRealmDiscoveryPolicy -IsOrganizationDefault $false

# 作成されたポリシーの確認
Write-Host "New Policy Created"
Write-Host $newPolicy

# Getting Service Principal
# 対象アプリ（Service Principal）の取得
$targetSP = Get-AzureADServicePrincipal -SearchString $targetSPName

# 対象アプリ（Service Principal）の確認
Write-Host "Target Service Principal"
Write-host $targetSP

# Assigning Policy to Service Principal
# 作成したポリシーを対象アプリ（Service Principal）に適用
Add-AzureADServicePrincipalPolicy -Id $targetSP.ObjectId -RefObjectId $newPolicy.Id

# Confirm Policy mapping to Service Principal
# 対象アプリ（Service Principal）に適用されたポリシーを確認
Get-AzureADServicePrincipalPolicy -Id $targetSP.ObjectId

# Unlink Policy from ServicePrincipal
# ServicePrincipalへのポリシー適用を解除
#Remove-AzureADServicePrincipalPolicy -id $targetSP.ObjectId

# Delete policy
# ポリシーの削除
#Remove-AzureADPolicy -id $policyId

```