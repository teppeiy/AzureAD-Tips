# ChromeやFirefoxでも統合Windows認証したい

### ADFS上で以下のPowerShellスクリプトを実行

https://blog.msresource.net/2016/11/18/adfs-iwa-and-the-wiasupporteduseragents-property/


```Powershell
Get-ADFSProperties | Select  -ExpandProperty WIASupportedUserAgents

$old=(Get-AdfsProperties).WIASupportedUserAgents

# for ADFS3.0 以下は次の2行もコメントアウトしてEdgeやWorkFolderもIWA対象とする
#$new=$old+"MS_WorkFoldersClient"
#$new=$old+"=~Windows\s*NT.*Edge"

$new=$old+"Mozilla/5.0"
Set-ADFSProperties -WIASupportedUserAgents $new
```