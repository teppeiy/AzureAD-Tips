# Web アプリケーションのモダン認証対応 (WIP)
## 概要
オンプレミスの時代から、クラウド利用が当たり前の自体になってきています。アプリケーションもどこからでも安全に利用できることが必須要件になりつつあり、Azure AD 等の IDaaS に認証連携し、様々なセキュリティや運用のメリットを享受するために認証もモダン化していくことを進めているかと思います。  
このドキュメントは、Web アプリケーションを対象に、アプリケーションの認証のモダン化の方法を整理します。iOS/Android/Windows にインストールするタイプのネイティブアプリケーションは対象にしません。

## モダン認証方式の定義
このドキュメントにおけるモダン認証方式の定義は、Azure AD や主要な IDaaS がサポートしているオープンな認証プロトコルを利用したもので、ネットワーク制限を受けないインターネット上で利用可能なものを指します。実質的には、以下２つの認証プロトコルを利用したものに絞られます（WS-Federation 対応のアプリケーションを作成することはほぼ無いため、対象外とします）。
* [Open ID Connect (+OAuth2)](https://www.openid.or.jp/document/index.html)
* SAML-P

逆に言うと、以下がモダン認証方式ではない方式の代表例です。このドキュメントでは、「オンプレミス認証方式」と呼びます。
* 統合 Windows 認証 (Kerberos、NTLM)
* LDAP 認証
* ヘッダー認証

## 推奨事項
Azure AD を ID プロバイダー とするアプリケーションを、できるだけコードを書かずに実装する、という観点での推奨を「新規に実装するアプリ」、「既存のアプリを」という2つの観点で考慮すべき順番を記載します。

オンプレミス認証方式 → モダン認証方式

新規アプリ
1. Azure App Service を利用する
2. MSAL、ADAL 等の認証ライブラリを利用して実装
3. ｘｘｘ

既存のアプリ
1. Azure App Service を利用する
2. ミドルウェアレベルで対応
3. MSAL、ADAL 等の認証ライブラリを利用して認証
4. Azure AD Application Proxy を利用する

## 対応方法
既存の「オンプレミス認証方式」アプリケーションを「モダン認証方式」へ移行し、Azure AD と認証連携する、という観点で対応方法を整理します。大きく分類すると、**アプリケーションのコード改修で対応する方法**と、**インフラ側で対応する方法**の2パターンがあります。

#### アプリコード改修で対応
| # | 移行方法  | コード改修工数 | コメント |
|---|---|---|---|
| A-1 | Azure App Service へ移行して PaaS 化 | 小～大 | 認証対応は、ほぼノンコーディングで実現可能。データをローカルディスク等に保存するアプリだとPaaS化改修が高くなる等、一般的な PaaS 化 の考慮点が適用される。 |
| A-2 | Visual Studio で認証設定を注入 | 小～大 | 既存アプリが VS で作成されている場合には非常に有効な方法で、シンプルな認証ロジックであれば、ほぼノンコーディングで実現可能。複雑な認可ロジックを実装している場合には要コード修正部分が多くなりがち。 |
| A-3 | MSAL/ADAL を組み込んでコード改修 | 中～大 | VS で作成されてないアプリケーションに対し、認証ライブラリを使って組み込む。シンプルな認証ロジックであれば、比較的少ないコード量で実現可能。複雑な認可ロジックを実装している場合には要コード修正部分が多くなりがち。 |
| A-4 | 自ら OIDC/SAML を実装してコード改修 | 大 | 整理のために記載するが、実質的にこの方法を選択することは無い。 |

#### インフラで対応（アプリコード改修は不要）
| # | 移行方法  | インフラ構成対応工数 | コメント |
|---|---|---|---|
| I-1 | ミドルウェアで対応 | 中～大 | 対応モジュールのある Apache にて利用可能 |
| I-2 | リバプロで対応 | 中～大 | 対応モジュールのある Apache にて利用可能 |
| I-3 | Azure AD Application Proxy を利用 | 小～中 | AppProxy用のコネクタサーバー等のインフラを用意 |


### 方法１：Azure App Service を利用する
[Azure App Service](https://docs.microsoft.com/ja-jp/azure/app-service/overview) を利用してアプリケーションそのものを PaaS 上で運用すると、Azure AD 認証対応も最小限のコードで非常に簡単に実装することが可能です。[Azure App Service](https://docs.microsoft.com/ja-jp/azure/app-service/overview) は、.NET、.NET Core、Java、Ruby、Node.js、PHP、Python　等の様々なプログラミング言語に対応しています。詳しくは、[Azure App Service での認証および認可](https://docs.microsoft.com/ja-jp/azure/app-service/overview-authentication-authorization) をご覧ください  
新しく作成する Web アプリケーションについてはもちろん、オンプレミス、または、IaaS のサーバー上で運用しているアプリケーションを移行することも検討することをお奨めします。


### 方法２：ミドルウェアレベルで対応
アプリケーション改修による費用対効果が見込めない場合には、改修工数が比較的低いミドルウェアレベルで対応できるか検討します。たとえば、Apache で利用できる [mod_auth_openidc](https://github.com/zmartzone/mod_auth_openidc) を利用することで、その Apache サーバーを OpenID Connect の Relying Party とすることができます。  


https://openid.net/developers/certified/

### 方法３：リバースプロキシを利用する
方法２の応用編になりますが、[mod_auth_openidc](https://github.com/zmartzone/mod_auth_openidc) を実装した Apache サーバーをリバースプロキシとして動作させ、既存の Webアプリケーションの前段に配置することで、既存の Web アプリケーションをほぼ改修することなく、OpenID Connect の Relying Party とすることが可能です。  
[OpenID ファウンデーションジャパンの Enterprise Idenity Wokring Group の公開しているドキュメント](https://eiwg.openid.or.jp/phase3/samples/implementation/mod_auth_openidc) をご参考ください。

### 方法４：Azure AD Application Proxy を利用する
方法３の応用編になりますが、リバースプロキシとして [Azure AD Application Proxy](https://docs.microsoft.com/ja-jp/azure/active-directory/manage-apps/application-proxy) を利用します。



### 参考情報
Azure AD 連携するアプリ開発についてのドキュメント、クイックスタート、サンプルコードについては、[開発者向け Azure Active Directory](https://docs.microsoft.com/ja-jp/azure/active-directory/develop/) をご覧ください。