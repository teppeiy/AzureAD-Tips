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

### User-Agentの例
Edge
```
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36 Edge/16.16299  
```

Chrome
```
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.186 Safari/537.36  
```  

FireFox  
```
Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:58.0) Gecko/20100101 Firefox/58.0  
```  

Microsoft Teams クライアント   
```
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Teams/1.1.00.5963 Chrome/59.0.3071.115 Electron/1.8.1 Safari/537.36  
```


## Firefoxへの信頼済みゾーンの追加
Firefoxは独自のゾーン設定を利用するため、ADFSにてIWAが利用できるよう、ADFSの社内URLを信頼済みゾーンに追加しておく必要があります。

https://blog.msresource.net/2015/12/11/ad-fs-enhanced-protection-for-authentication-epa-chrome-and-integrated-windows-authentication-iwa/

* network.negotiate-auth.trusted-uris  
* network.automatic-ntlm-auth.trusted-uris

![Firefox Zone Configuration](https://msresource.files.wordpress.com/2015/12/epafirefoxiwasettingsview.png)


## 以下も参考
https://docs.microsoft.com/ja-jp/windows-server/identity/ad-fs/operations/configure-intranet-forms-based-authentication-for-devices-that-do-not-support-wia
