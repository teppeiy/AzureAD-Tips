# ChromeやFirefoxでも統合Windows認証したい

## ADFS上で以下のPowerShellスクリプトを実行

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


## Firefoxへの信頼済みゾーンの追加
Firefoxは独自のゾーン設定を利用するため、ADFSにてIWAが利用できるよう、ADFSの社内URLを信頼済みゾーンに追加しておく必要があります。

https://blog.msresource.net/2015/12/11/ad-fs-enhanced-protection-for-authentication-epa-chrome-and-integrated-windows-authentication-iwa/

* network.negotiate-auth.trusted-uris  
* network.automatic-ntlm-auth.trusted-uris

![Firefox Zone Configuration](https://msresource.files.wordpress.com/2015/12/epafirefoxiwasettingsview.png)