## インフラストラクチャー、運用
| オプション  | パスワード同期 + sSSO  | パススルー認証 + sSSO  |  AD FS |
|---|---|---|---|
|必要サーバー数|最低 1 台、推奨 2 台（ステージングモード）|	最低 1 台、推奨 2 台（エージェント冗長化）|	最低 1 台、推奨 2 台（サーバーファーム）|
|社外アクセス時のDMZのサーバー数|不要|	不要|	最低 : 1 台、推奨 : 2 台（高可用性）|
|自動フェールオーバーのサポート See note 1|いいえ|はい|はい|
|SSL証明書が必要|いいえ|いいえ|はい|
|SCOMでのオンプレミスコンポーネントの監視|いいえ|いいえ|はい|
|オンプレミスコンポーネントの監視|Connect Health (Premium P1)|一部|Connect Health (Premium P1)|

## 認証
| オプション  | パスワード同期 + sSSO  | パススルー認証 + sSSO  |  AD FS |
|---|---|---|---|
|ADパスワードサインイン|はい|はい|はい|
|ドメイン参加PCからSSO|	はい|	はい|	はい|
|AD - Soft Certificates (MDM or GPO provisioned)|いいえ|いいえ|はい|
|スマートカード認証  |  いいえ	|いいえ|	はい|
|ADアカウント無効化に伴うAzureADアカウント無効化のタイミング | 同期サイクルである30分後（既定）に反映される |即座|即座|
|ADアカウントロックアウト時のAzureADアカウント認証可否|可|否|否|
|ADアカウントのパスワード失効時のAzureADアカウント認証可否|可|否|否|
|信頼関係がある複数のAD フォレストのユーザーの認証|はい|	はい|	はい|
|信頼関係がない複数のAD フォレストのユーザーの認証 see note 3|はい|	いいえ	|はい（AD FS 2016）|
|サードパーティーの LDAP ディレクトリによるサインインSee note 4|いいえ|いいえ|はい(AD FS 2016)|
|Authenticator Appをプライマリ認証として利用（パスワードレス認証）|	いいえ|いいえ|はい(AD FS 2016)|

## 多要素認証(MFA)
| オプション  | パスワード同期 + sSSO  | パススルー認証 + sSSO  |  AD FS |
|---|---|---|---|
|Azure MFA(SMS、電話、ワンタイムパスコード)|はい|はい|はい|
|MFA Server(PINモード & H/Wトークン)|いいえ|いいえ|はい|
|Win10 の Windows Hello for Business（キー ベース）|はい|	はい|はい（AD FS 2016）|
|Win10 の Windows Hello for Business（証明書ベース）see note 5|はい（with MDM）|はい（with MDM)|はい|
|サードパーティーMFA連携 see note 6|	はい(Premium P2)|	はい(Premium P2)|はい|
|カスタムMFAプロバイダ|いいえ|いいえ|はい|

## アプリケーション
| オプション  | パスワード同期 + sSSO  | パススルー認証 + sSSO  |  AD FS |
|---|---|---|---|
|ブラウザー|はい|はい|はい|
|Exchange Active Sync (EAS)	|はい|はい|はい|
|ネイティブアプリ (レガシー認証)|はい|Coming Soon	|はい|
|ネイティブアプリ (モダン認証)|はい|はい|はい|
|Win10 PCサインイン with U/P on AzureAD参加デバイス|はい|Coming Soon|はい|

## サインインエクスペリエンス
| オプション  | パスワード同期 + sSSO  | パススルー認証 + sSSO  |  AD FS |
|---|---|---|---|
|サインイン ページのカスタマイズ|はい(Premium P1)|はい(Premium P1)|はい|
|CSS や JavaScript によるカスタマイズ|いいえ|いいえ|はい|
|UPNを使ったサインイン|はい|はい|はい|
|Domain\sAMAccountNameを使ったサインイン	see note 7|いいえ|いいえ|はい|
|Seamless first time sign-in to O365 native apps on Domain Joined devices inside corp network (includes Office Pro Activation)See note 8|いいえ|いいえ|はい|
|Seamless 2nd time sign-in to O365 native apps on Domain Joined devices|はい|はい|はい|

## パスワード失効時の通知・変更
| オプション  | パスワード同期 + sSSO  | パススルー認証 + sSSO  |  AD FS |
|---|---|---|---|
|パスワード期限切れ通知|いいえ|いいえ|はい|
|Custom password change URL link shown in Office Portal & Win10 desktop|いいえ|いいえ|はい|
|Integrated password change experience when user’s password has expired|いいえ|いいえ|はい|


## デバイスとアクセスコントロール
| オプション  | パスワード同期 + sSSO  | パススルー認証 + sSSO  |  AD FS |
|---|---|---|---|
|デバイス登録 : ドメイン参加Win10|はい|はい	|はい|
|デバイス登録 : ドメイン参加Win7/8.1|はい	|はい	|はい|
|レガシープロトコルのブロック	|Coming Soon (Premium)|	Coming Soon (Premium)|	はい|
|イントラネットからのみレガシープロトコルの許可（Office 2010 など）|Coming Soon (Premium)|Coming Soon (Premium)|はい|


## その他
| オプション  | パスワード同期 + sSSO  | パススルー認証 + sSSO  |  AD FS |
|---|---|---|---|
|セルフサービスパスワードリセット(書き戻し)	|はい(Premium P1)|はい(Premium P1)|はい(Premium P1)|
|オンプレミスのパスワードポリシーの適用|一部（パスワードの複雑性のポリシー、パスワードの有効期限のポリシー）|	はい|	はい|
|オンプレミスユーザーアカウントのロックアウト保護|-|スマートロックアウト|エクストラネットロックアウト|
|Azure AD の条件付きアクセス|はい|はい|はい|


## Notes
1. HA with auto-failover: In the case of PTA, this is automatically done by the cloud service. In the case of ADFS, the load balancer can be configured to send requests only to ‘healthy’ nodes. With password # sync, you will have to manually elevate the staging Azure AD Connect server to become active if the current server goes down.
2. Certificate sign-in: ADFS can integrate with your enterprise PKI to allow sign-in using certificates. These certificates can be soft-certificates deployed via trusted provisioning channels such as MDM or GPO or smartcard certificates (including PIV/CAC cards) or Hello for Business (cert-trust). See this blog for more information.
3. Users in multiple untrusted AD forests: With ADFS in 2016, an untrusted forest can be configured as an LDAP directory that allows you to sign-in those users from a single ADFS setup. To learn more about LDAP directory setup with ADFS, see this link.
4. 3rd party LDAP directory: Customers requiring to sign-in users from a 3rd party LDAP directory to access Azure AD apps (and Office 365) can now use a new feature in ADFS 2016 to sign these users in. To learn more about this feature and configure it, see this link.
5. Win10 Hello For Business (Cert Trust): For customers looking to deploy Hello For Business as part of their Win10 deployment, this option is only relevant if you need the TPM protected credentials to be used with down-level domain controllers (<2016 DC’s) or with VPNs that only support certificates. Otherwise, the key trust model can be used. Also, in this option, ADFS 2016 also becomes the provisioning endpoint for the TPM protected certificates and makes the provisioning simpler with auto-renewal & instant PIN use (coming soon). It is also possible to use MDM to provision the certificates. For Hybrid Azure AD Joined devices (domain joined devices that are also registered with Azure AD), the ADFS option is recommended due to the more seamless and deterministic experience that is integrated with Windows logon.
6. Custom MFA: Azure AD recently announced support for 3rd party MFA providers integrated with Conditional Access. This is currently in preview and supports RSA, Trusona & Duo. To learn more see this link.
7. Sign-in with domain\samaccountname: While ADFS supports this and can be used in multiple use cases, we recommend customers to move to signing in with a UPN suffix. Also, it is possible in ADFS in a single UPN suffix scenario to customize the Java Script in the sign-in pages to only require the samaccountname.
8. Office first time login on DJ devices: Office apps are optimized on first sign-in to look up the local UPN and seamlessly sign-in the user using WS-Trust Kerberos endpoints from the Identity provider. If this is not available, Office throws up a dialog where the user must enter their UPN in the Azure AD home realm discovery page which then redirects to the IDP and can support desktop SSO. This capability is likely needed if you are planning on exposing Office apps through a Virtual Desktop Solution that is not using persistent sessions. In this case, each login is treated like a first time login.