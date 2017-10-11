# ADFSのログイン画面にSSPRへのリンクを表示

contoso.com部分は、御社のドメイン名（Azure ADに登録済みのもの）を指定してください。
```Powershell
Set-ADFSGlobalWebContent -SigninPageDescriptionText "<p><A href=https://passwordreset.microsoftonline.com?whr=contoso.com>Can’t access your account?<br>アクセスできませんか？<A/></p>"
```

https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-passwords-customize