# ADFS廃止のすすめ

## 概要
これまで、Office 365等へのシングルサインオンやアクセス制御をAD FSで実施するというケースが主流でしたが、最近ではAzure ADへのシフトが加速化されています。AD FSを利用しないメリットとして以下のような点が挙げられます。
1. よりインテリジェントなセキュリティ対策のメリットを享受  
AD FSに対する攻撃対策から解放され、Azure ADのインテリジェントなセキュリティ対策のメリットを享受することができます。たとえば、[Smart Lockout](https://docs.Microsoft.com/ja-jp/azure/active-directory/active-directory-secure-passwords#azure-ad-password-protections)を利用することになり、様々な攻撃から防御されます。
2. [条件付きアクセス](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-conditional-access-azure-portal)で、より高度できめ細かなアクセス制御を実現  
複雑な構文を要するAD FSのクレームルールを管理することは手間がかかりますし、変更管理やテストにかかる労力や変更適用までの時間は小さくありません。条件付きアクセスを利用することで、GUIを使った分かりやすい形でのアクセスポリシーを管理できます。

3. オンプレミス環境の管理から解放  
ADFSを利用する場合、HA/DRを考慮すると最低4台（AD FS/WAP x 2）また、テスト・QA環境も含めると更に多くのサーバーの管理が必要です。そして、各種証明書の管理、DMZを含む外部からのアクセスセキュリティの管理、ロードバランサーの管理、セキュリティ更新プログラムの管理等、比較的大きなコストと労力が必要ですが、これから開放されます。

## AD FS撤廃までのステップ
1. 準備
    * パスワードハッシュ同期の有効化
    * AD FSに依存しているサービスを移行
        * アプリケーション（証明書利用者信頼）
        * デバイス登録サービス
2. 認証の切り替え
3. 廃止


## パスワードハッシュ同期の有効化
パスワードハッシュ同期は、AD FSを無くすためのステップの一つですが、2つのメリットをもたらします。
1. [漏洩した資格情報検知レポート](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-identityprotection#users-flagged-for-risk)によるセキュリティ向上
マイクロソフトは、複数のソースから漏洩した資格情報一覧を継続的に取得しています。そのリストとAzure ADアカウントと機械的に突き合わせることにより、漏洩した資格情報を検知しレポートを提供します。
2. ディザスタリカバリへの対策
フェデレーション環境において、AD FS/WAPが利用できなくなった際には、Office 365等Azure ADに認証を依存しているサービスが利用できなくなります。実際にAD FSがPetyaに感染した企業において、パスワードハッシュ同期をしていた企業はAD FSからAzure ADへの認証に切り替えることによって、数時間のダウンタイムでサービスへの認証アクセスが復旧できました。一方、パスワードハッシュ同期をしていなかったため、AD FSを構築しなおす等の作業で、数日間のダウンタイムを強いられた企業もあります。

#### パスワードハッシュをAzure ADに同期することの安全性について
パスワードハッシュと言えども、Azure ADに資格情報を同期することに懸念を持つ方々も少なくありません。その場合、同期プロセスや[Smart Lockout](https://docs.Microsoft.com/ja-jp/azure/active-directory/active-directory-secure-passwords#azure-ad-password-protections)の説明が助けになるかもしれません。  

#### パスワードハッシュ同期のメカニズム
パスワードハッシュ同期を有効化すると、Azure AD Connectは、元々のパスワードハッシュをソルト+ストレッチング+ハッシュといったテクニックを使い、攻撃リスクを軽減するプロセスを通してAzure ADへ同期します。
* ソルト: ハッシュ化前に対象文字列にランダムな文字列を不可する。レインボーテーブルによる探索に対する防御策の一つ
* ストレッチング: 何度もハッシュ化をすることにより、攻撃者の解析コストを高める（時間の長期化）防御策の一つ


    参考：
    [ハッシュとソルト、ストレッチングを正しく理解する：本当は怖いパスワードの話](http://www.atmarkit.co.jp/ait/articles/1110/06/news154.html)


具体的には、ADに保存されているMD4でハッシュ化されたパスワードにsaltを付加、[PBKDF2](https://www.ietf.org/rfc/rfc2898.txt)関数で計算後、[HMAC-SHA256](https://msdn.microsoft.com/library/system.security.cryptography.hmacsha256.aspx)
を使ってキー付きハッシュアルゴリズムで1,000回ものハッシュ化を実施する、といったような一連のプロセスです。詳しくは[こちらのドキュメント](https://docs.microsoft.com/ja-jp/azure/active-directory/connect/active-directory-aadconnectsync-implement-password-synchronization#how-password-synchronization-works)を参照ください。  

#### 手順:
1. [Azure AD Connectの構成ウィザードを利用し、パスワードハッシュ同期を有効化する](https://docs.microsoft.com/ja-jp/azure/active-directory/connect/active-directory-aadconnectsync-implement-password-synchronization#enable-password-synchronization)
2. [漏洩した資格情報検知レポートを確認する](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-reporting-security-user-at-risk)

#### Azure AD のパスワード保護
* Smart Lockout
ユーザーのパスワードがサイバー犯罪者によってハッキングされようとしている可能性があることを Azure AD が検出すると、Microsoft は、[Smart Lockout](https://docs.Microsoft.com/ja-jp/azure/active-directory/active-directory-secure-passwords#azure-ad-password-protections) を使ってそのユーザー アカウントをロックします。

#### 更にセキュリティレベルを上げる
* [Identify Protection](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-identityprotection) によるリスクベースでの条件付きアクセスの有効化


参考:  
Ignite 2017 - [Shut the door to cybercrime with Azure Active Directory risk-based identity protection](https://myignite.microsoft.com/sessions/53404?source=sessions)

## AD FSに依存しているサービスを移行
### 移行可否の判断のための観点
* アプリケーション (証明書利用者信頼)   
AD FSをIdentity Providerとしているアプリケーションを棚卸しします。そしてRelying Party毎のアクセス制御方法 (クレームルール) がAzure ADで実装可能か確認します。
* 認証方式   
AD FSでしか利用することのできない認証方式が必要か確認します。
* ライセンス   
移行に際して必要なライセンスを保持しているか確認します。

### アプリケーション (証明書利用者信頼)   
現状の棚卸し、移行判断にはAzure AD開発部門で開発した[AD FS Config Dump](ADFS-Config-Dump.md)により取得したAD FSの設定情報から、証明書利用者信頼の移行判定レポートを提供します。

#### よくある移行障壁
1. EncryptClaims
トークンを暗号化している状態にあります。現状Azure ADではサポートされていない機能です。本当にトークンの暗号化が必須要件であるか再度検討をお奨めいたします。
また、Azure ADでも近々トークン暗号化をサポートする予定です。
2. IssuanceAuthorizationRule（クレーム発行許可ルール）
いわゆるクレームルールで、アクセス制御している状態にあります。Azure ADの条件付きアクセスを利用することで、更に柔軟で管理のしやすいアクセス制御をすることが可能です。移行における大部分の労力はクレームルール⇒条件付きアクセスの移行に費やされることになります。

#### AD FSでしか実現することができない要件の確認
Azure ADの継続的な機能向上に伴い、AD FSでしか実現できないシナリオは急速に減りつつありますが、まだ、いくつか気を付けるべきポイントが存在します。
* グループクレーム
* 複雑なクレーム発行ルール  
* ・・・   

#### クレームルール から[条件付きアクセス](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-conditional-access-azure-portal)へ移行
メリット
* より柔軟なアクセス制御
* リスクベースのアクセスの適用(要P2)   

#### 考慮ポイント
* レガシー認証のブロック → 近い将来実現予定
* レガシー認証の条件付きアクセス → 近い将来実現予定
* クライアント証明書を使った認証

詳しくは、[ハイブリッドID認証方式の比較](Hybrid-Comparison.md)を参照ください。


### ライセンス
AD FSでクレームルールを利用し、場所ベースのアクセス制御をしている利用ケースは少なくありません。これと同様あるいは更に高度な機能な有する条件付きアクセスを利用することになりますが、Azure AD Premium P1ライセンスが必要になります。これまで、ITシステムの利用が限られているユーザー（例：工場勤務の従業員等）がライセンスを保持していない場合には、追加購入の検討が必要になります。
マイクロソフトの営業担当者へご相談ください。

#### 移行戦略
* 移行対象グループがAD FSの（クレームルールによる）アクセス制御の影響を受けず、条件付きアクセスを利用している状態を確立し、移行対象グループのメンバーを徐々に増やしていく
* クレームルールがバイパスされるGroup Aがあるため、その配下に「CA Users（仮称）」グループを作成し、そのグループを条件付きアクセスの対象グループとする
##### この方式の問題点
オンプレADでCA Usersグループに対象アカウントを登録し、Azure AD Connectでグループメンバーシップ情報がAzure ADへ同期されるまでの間（最大30分程度、同期周期に依存）、AD FSのクレームルールはバイパス、条件付きアクセスの対象にならない（＝アクセス許可）という状態が存在する
リスク軽減方法 => CA Usersグループにメンバーを追加した直後に同期をする等

