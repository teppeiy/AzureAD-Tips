# 条件付きアクセスのFAQ

## 概要
条件付きアクセスについて、よく聞かれる質問と回答を記載します。


### Q1. マイクロソフト以外のアプリが、条件付きアクセスでブロックされます
モバイルデバイス（iOS/Android）上のネイティブクライアント（アプリ等）を利用したデバイスベースの条件付きアクセスは、マイクロソフトのアプリのみ対応している、と考えておいたほうが良いかもしれません。  
  
たとえば、[Salesforce1](https://itunes.apple.com/us/app/salesforce/id404249815?mt=8)等のネイティブアプリを利用する場合、「デバイスは準拠しているとしてマーク済みである必要があります」 という制限付きアクセス許可ルール対象である場合、アクセスはブロックされます。これは、[Salesforce1](https://itunes.apple.com/us/app/salesforce/id404249815?mt=8)（アプリ）が、デバイス証明書（iOSの場合キーチェーンに格納）にアクセスし、その証明書をAzure ADへ提示する（ことでアクセス元デバイス情報を伝える）必要があるのですが、モバイルOSのセキュリティ仕様上、デバイス証明書へアクセスできるアプリは限定されています。  
  
アプリがブローカー認証（代理認証のメカニズム）を実装することで、デバイスベースの条件付きアクセスが利用可能です。Microsoft Authenticator等が代理認証をすることで、デバイス証明書を取得し、それを通じてAzure ADへデバイス情報を伝える事が可能になります。  
* [iOS で ADAL を使用してクロス アプリ SSO を有効にする方法](https://docs.microsoft.com/ja-jp/azure/active-directory/develop/active-directory-sso-ios)

* [Android で ADAL を使用してクロス アプリ SSO を有効にする方法](https://docs.microsoft.com/ja-jp/azure/active-directory/develop/active-directory-sso-android)


**デバイスベースの条件付きアクセスを利用したいアプリがそれに対応していない場合、利用企業よりアプリベンダーへ対応要望いただくことをお願いしています。**
  
  
上記した制限事項は、ネイティブクライアントでデバイスベースの条件付きアクセスを利用するケースに限ります。  
* [サポートされているブラウザ（デバイス証明書にアクセス可能）](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-conditional-access-technical-reference#client-apps-condition)はデバイスベースの条件付きアクセスを利用可能です
* 「多要素認証を要求する」場合には、ブローカー認証に対応していないネイティブクライアントからも利用可能です

デバイスベースの条件付きアクセス とは、以下のアクセス制御が設定された条件付きアクセスのことを指します。
* デバイスは準拠しているとしてマーク済みである必要があります
* ドメイン参加済みであることが必要 (ハイブリッド Azure AD) 


### その他のよくある質問

[Japan Azure Identity Support Blog](https://blogs.technet.microsoft.com/jpazureid/) より [Azure AD の条件付きアクセスに関する Q&A](https://blogs.technet.microsoft.com/jpazureid/2017/12/04/conditional-access-qa/)もご参考ください。