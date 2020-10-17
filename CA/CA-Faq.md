# 条件付きアクセスのFAQ

## 概要
条件付きアクセスについて、よく聞かれる質問と回答を記載します。


### Q1. マイクロソフト以外のアプリが、条件付きアクセスでブロックされます
モバイルデバイス（iOS/Android）上のネイティブクライアント（アプリ等）を利用したデバイスベースの条件付きアクセスは、マイクロソフトのアプリのみ対応している、と考えておいたほうが良いかもしれません。  
  
たとえば、[Salesforce1](https://itunes.apple.com/us/app/salesforce/id404249815?mt=8)等のネイティブアプリを利用する場合、「デバイスは準拠しているとしてマーク済みである必要があります」 という制限付きアクセス許可ルール対象である場合、アクセスはブロックされます。これは、[Salesforce1](https://itunes.apple.com/us/app/salesforce/id404249815?mt=8)（アプリ）が、デバイス証明書（iOSの場合キーチェーンに格納）にアクセスし、その証明書をAzure ADへ提示する（ことでアクセス元デバイス情報を伝える）必要があるのですが、モバイルOSのセキュリティ仕様上、デバイス証明書へアクセスできるアプリは限定されています。  
  
#### 解決方法
1. [Apple デバイス用の Microsoft Enterprise SSO プラグイン](https://docs.microsoft.com/ja-jp/azure/active-directory/develop/apple-sso-plugin)を利用する  
iOS 向けの SSO プラグインを利用することで、ブローカーや MSAL を利用していないアプリでもデバイスベースの条件付きアクセスが利用可能になります。利用のための[必要条件](https://docs.microsoft.com/ja-jp/azure/active-directory/develop/apple-sso-plugin#requirements)ご確認ください。

2. アプリがブローカー認証（代理認証のメカニズム）を実装することで、デバイスベースの条件付きアクセスが利用可能です。Microsoft Authenticator等が代理認証をすることで、デバイス証明書を取得し、それを通じてAzure ADへデバイス情報を伝える事が可能になります。  
* [iOS で ADAL を使用してクロス アプリ SSO を有効にする方法](https://docs.microsoft.com/ja-jp/azure/active-directory/develop/active-directory-sso-ios)

* [Android で ADAL を使用してクロス アプリ SSO を有効にする方法](https://docs.microsoft.com/ja-jp/azure/active-directory/develop/active-directory-sso-android)

3. System WebViewに対応することでも実現可能です。ブローカー認証のように、クロス アプリ SSOはできませんが、比較的簡単なアプリ変更でこちらは実装可能です。

4. アプリ独自の設定で対応
Salesforce には、証明書ベースの認証を有効にするという設定があります。詳しくは、[こちら](https://help.salesforce.com/articleView?id=000340176&language=ja&type=1&mode=1)をご参考ください。


デバイスベースの条件付きアクセス とは、以下のアクセス制御が設定された条件付きアクセスのことを指します。
* デバイスは準拠しているとしてマーク済みである必要があります
* ドメイン参加済みであることが必要 (ハイブリッド Azure AD) 


### Q2. Intuneへの登録が失敗します
条件付きアクセスで、**全て**のクラウドアプリへのアクセスを**ブロック**、というポリシーが適用されている場合、Intuneへの登録が失敗します。
"Microsoft Intune Enrollment"クラウドアプリを対象外にしていても登録が失敗します。  
これは、Intuneへの登録プロセスにおいて、Intune Company Portalアプリが"Microsoft Intune Enrollment"以外のクラウドアプリへアクセスする必要があるからです。そのクラウドアプリは、条件付きアクセスの対象外アプリとして選択することができません。

対処方法：全てのクラウドアプリへのアクセスを**ブロック**する代わりに、**デバイスは準拠しているとしてマーク済みである必要があります**を利用します。この場合、"Microsoft Intune Enrollment"クラウドアプリを対象外に設定せずともIntuneの登録は可能です。

### Q3. 条件が合っているはずなのに、アクセス制御が利かない
レガシー認証を利用するプロトコル（POP3/IMAP等）でOutlookからExchange Onlineへアクセスする際、そのプロトコルではOS情報が必ずしも付帯されて送信されません。こういったことから、Azure ADはOS情報を知る術がないということになり、「デバイスプラットフォーム」条件に合致せず、意図するポリシーが適用されないことがあります。

対象方法としては、デバイスプラットフォーム条件にて「すべてのプラットフォーム (サポート対象外を含む)」を利用し、OS情報が認識されない場合にもポリシーが適用される（条件が合致する）ようにします。レガシー認証関連のポリシーのみではなく、一般的な条件付きアクセスポリシーを利用する際の一般的なベストプラクティスです。

「デバイスプラットフォーム」条件を100％信頼したポリシーデザインは、セキュリティ観点で好ましくありません。たとえば、利用者がアクセス元User-Agentを偽り、OS情報をご認識させてIT管理者が意図するポリシーをバイパスされてしまう可能性はゼロではありません。条件付きアクセスのOS情報の認識方法は、Azure ADが認識できるあらゆる情報（非公開）を使うことで精度を高めていますが、前述したようなリスクがゼロではないということをご認識ください。

### その他のよくある質問

[Japan Azure Identity Support Blog](https://blogs.technet.microsoft.com/jpazureid/) より [Azure AD の条件付きアクセスに関する Q&A](https://blogs.technet.microsoft.com/jpazureid/2017/12/04/conditional-access-qa/)もご参考ください。
