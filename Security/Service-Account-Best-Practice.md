# サービスアカウント の セキュリティプラクティス

## 概要
Azure AD で認証保護された Microsoft Graph、API、他サービスにアクセスするアプリケーションやバッチプログラムを運用する際のサービスアカウントについてのセキュリティプラクティスです。  

## 避けるべきパターン
アクセス元のアプリケーションやバッチプログラムに、ユーザーアカウントの ID、パスワードを埋め込むというやり方をされているケースをよく見かけますが、これは、アカウントの漏洩リスクが高くなるため、お奨めできません。  
たとえば、ソースコードや構成ファイルをソースコード管理ツールで共有してしまうと、意図しない他者にアカウント情報が漏れてしまいます。また、ユーザーアカウントを利用すると、対話的ログインが利用できないため、多要素認証等を利用したアカウント保護をすることが困難です。さらに、こういった類のアカウントに特権（例：ユーザー管理者、全体管理者等）を与えていることが多いですが、その場合には漏洩時の影響は非常に大きくなります。 

まとめると、  
**ユーザーの資格情報（ユーザー・パスワード）をプログラムに埋め込むべきではない**、なぜなら、
* 資格情報（ID/パスワード）が漏洩しやすい
* 対話的ログインで利用できるような方法（例：多要素認証）でのアカウント保護が難しい
* それが特権アカウントであれば、漏洩時の影響が多大である

## ベストプラクティス
### 1.  [Managed Identity](https://docs.microsoft.com/ja-jp/azure/active-directory/managed-identities-azure-resources/overview) を利用する
アクセスされる側のリソースが Azure 上で運用されている場合には、Managed Identity を利用します。この仕組を利用することで、アプリケーションに資格情報を埋め込むことなく安全にリソースアクセスが可能です。
   
### 2. [サービスプリンシパル](https://docs.microsoft.com/ja-jp/azure/active-directory/develop/app-objects-and-service-principals) を利用した証明書認証を利用する
他のクラウドサービス（AWS 等）にアクセスされる側のリソースが運用されている場合には Managed Identity が利用できないため、サービスプリンシパルを利用します。  
サービスプリンシパルを認証する方式がいくつか選択できますが、[証明書を利用した方式](https://docs.microsoft.com/ja-jp/azure/active-directory/develop/howto-authenticate-service-principal-powershell)を利用します。
ClientId/Secret での認証も可能ですが、これらをアプリケーションに埋め込むことは、ユーザーアカウントを利用して資格情報（ID/パスワード）を埋め込むことと同様に、漏洩リスクが高いため、これは避けるべき方法です。

参考：[3 分でわかる Azure での Service Principal](https://www.slideshare.net/ToruMakabe/3azure-service-principal)

### 3. やむなくユーザーアカウントを利用する場合
アプリケーションの要件によっては、ユーザーアカウントを利用しないとならないケースがあります。たとえば、Exchange Online の（すべてではなく）**一部**のリソースメールボックスを制御するプログラム、という要件がある場合等です。サービスプリンシパルに対して与えることの権限の粒度は、Exchange Online にある全てのリソースメールボックスであるため、一部のリソースメールボックスへの権限を与えるには、ユーザーアカウントを利用せざるをえません。  

その場合においては、
* 最小の権限を与えて運用する（全体管理者を与える必要性は殆ど無いはずです）
  * [Azure AD で利用できるロール](https://docs.microsoft.com/ja-jp/azure/active-directory/users-groups-roles/directory-assign-admin-roles) を参考に、アプリケーションの要件に必要最小限のロールを与えます
* パスワードを定期的に変更する
  * アプリケーション側の設定変更（資格情報の更新）も必要になりますので、運用負荷は高くなりがちですが、資格情報漏洩時の影響を最小限にするためには必要な運用です
* [条件付きアクセス](https://docs.microsoft.com/ja-jp/azure/active-directory/conditional-access/overview) を利用して、アクセス元の場所制限や対象アプリケーションを絞る等の対策で、アタックサーフェスを最小化する
* アカウントのアクティビティを監視する
  * [サインインログ](https://docs.microsoft.com/ja-jp/azure/active-directory/reports-monitoring/concept-sign-ins) を利用して対象アカウントのサインインイベントを確認することが可能です。また、SIEMへログを転送して監視の統合・自動化をすることも可能です。
  * [Azure Monitor との連携](https://docs.microsoft.com/ja-jp/azure/active-directory/reports-monitoring/concept-activity-logs-azure-monitor) することで、任意の条件にを満たしたサインインイベント発生時にアラートをするというような仕組みを構築することも可能です。