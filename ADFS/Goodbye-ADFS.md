# ADFS廃止のすすめ

## 概要
これまで、Office 365等へのシングルサインオンやアクセス制御をAD FSで実施するというケースが主流でしたが、最近ではAzure ADへのシフトが加速化されています。AD FSを利用しないメリットとして以下のような点が挙げられます。

* **よりインテリジェントなセキュリティ対策のメリットを享受**  
AD FSに対する攻撃対策から解放され、Azure ADのインテリジェントなセキュリティ対策のメリットを享受することができます。たとえば、[Smart Lockout](https://docs.Microsoft.com/ja-jp/azure/active-directory/active-directory-secure-passwords#azure-ad-password-protections)を利用することになり、様々な攻撃から防御されます。
* [**条件付きアクセス**](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-conditional-access-azure-portal)**で、より高度できめ細かなアクセス制御を実現**  
複雑な構文を要するAD FSのクレームルールを管理することは手間がかかりますし、変更管理やテストにかかる労力や変更適用までの時間は小さくありません。条件付きアクセスを利用することで、GUIを使った分かりやすい形でのアクセスポリシーを管理できます。
* **オンプレミス環境の管理から解放**  
AD FSを利用する場合、HA/DRを考慮すると最低4台（AD FS/WAP x 2）また、テスト・QA環境も含めると更に多くのサーバーの管理が必要です。そして、各種証明書の管理、DMZを含む外部からのアクセスセキュリティの管理、ロードバランサーの管理、セキュリティ更新プログラムの管理等、比較的大きなコストと労力が必要ですが、これから開放されます。

## AD FS撤廃までのステップ
1. 準備  
    1.1. パスワードハッシュ同期の有効化  
    1.2. AD FSに依存しているサービスを移行  
    * アプリケーション（証明書利用者信頼）
    * デバイス登録サービス
2. 認証の切り替え
3. 廃止


## 1.1. パスワードハッシュ同期の有効化
パスワードハッシュ同期は、AD FSを無くすためのステップの一つですが、2つのメリットをもたらします。
1. [漏洩したID検知レポート](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-identityprotection#users-flagged-for-risk)によるセキュリティ向上
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

## 1.2. AD FSに依存しているサービスを移行
### 移行可否の判断のための観点
* アプリ (証明書利用者信頼) 毎のクレームルール    
AD FSをIdentity Providerとしているアプリケーションを棚卸しします。そして、証明書利用者信頼毎の発行変換ルール(Issuance Transformation Rules)がAzure ADで実装可能か確認します。また、発行承認ルール(Issuance Authorization Rules)は、Azure ADの条件付きアクセスへ移行します。
* ライセンス   
移行に際して必要なライセンスを保持しているか確認します。
* デバイス登録サービス (DRS)  
AD FSのデバイス登録サービスを利用している場合には、Azure AD DRSへ移行します。

### アプリ (証明書利用者信頼)の棚卸し  
移行判断にはAzure AD開発部門で開発した[AD FS Config Dump](ADFS-Config-Dump.md)により取得したAD FSの設定情報から、証明書利用者信頼の移行判定レポートを提供します。

#### 考慮ポイント
* トークンに関する要件、発行変換ルール  
Azure ADの継続的な機能向上に伴い、AD FSでしか実現できないシナリオは急速に減りつつあります。
    * 最近のアップデートで、トークンの暗号化がサポートされています
    * 近日中に、SAML1.1トークンがサポートされます
    * 近日中に、ToLowercaseやRegexReplaceを利用した動的なクレーム発行もされます

* クレーム発行許可ルール  
アクセス制御をAzure ADの条件付きアクセスを利用することで、更に柔軟で管理のしやすいアクセス制御をすることが可能です。移行における大部分の労力はクレームルール⇒条件付きアクセスの移行に費やされることになります。

### アクセス制御：承認要求規則から[条件付きアクセス](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-conditional-access-azure-portal)へ移行
メリット
* より柔軟なアクセス制御
* リスクベースの条件付きアクセスの適用(要P2)  

#### 考慮ポイント
レガシープロトコルの条件付きアクセス対応 → 近い将来実現予定  
現状、Office2010等、レガシープロトコルを利用するトラフィックは、Azure ADの条件付きアクセスの対象外となり、AD FSでしか制御することができません。メールプロトコル(POP/IMAP/ActiveSync)については、Exchange Onlineのポリシーで有効化・無効化することが可能です。

#### ライセンス
AD FSでクレームルールを利用し、場所ベースのアクセス制御をしている利用ケースは少なくありません。これと同様あるいは更に高度な機能な有する条件付きアクセスを利用することになりますが、Azure AD Premium P1ライセンスが必要になります。これまで、ITシステムの利用が限られているユーザー（例：工場勤務の従業員等）がライセンスを保持していない場合には、追加購入の検討が必要になります。
マイクロソフトの営業担当者へご相談ください。

#### [Azure AD の条件付きアクセスに関する Q&A](https://blogs.technet.microsoft.com/jpazureid/2017/12/04/conditional-access-qa/)

#### アクセス制御の移行戦略
* 移行対象グループがAD FSの（クレームルールによる）アクセス制御の影響を受けず、条件付きアクセスを利用している状態を確立し、移行対象グループのメンバーを徐々に増やしていく
* クレームルールがバイパスされるGroup Aがあるため、その配下に「CA Users（仮称）」グループを作成し、そのグループを条件付きアクセスの対象グループとする
##### この移行方式の問題点
* オンプレADでCA Usersグループに対象アカウントを登録し、Azure AD Connectでグループメンバーシップ情報がAzure ADへ同期されるまでの間（最大30分程度、同期周期に依存）、AD FSのクレームルールはバイパス、条件付きアクセスの対象にならない（＝アクセス許可）という状態が存在する  
    * リスク軽減方法 => CA Usersグループにメンバーを追加した直後に同期をする等  
* レガシー認証をブロックする等、既存のルールの影響を受けなくなるため、たとえば、自宅PCのOutlook2010からExchage Onlineにアクセスできてしまう  
    * CA Usersグループを対象としたレガシー認証コントールのクレームルールを追加する

### デバイス登録サービス (DRS)  
Azure AD DRS へ移行します。  
XXX 


## 2. 認証切り替え
パスワードハッシュ同期の有効化、AD FSに依存しているアプリ(証明書利用者信頼)やアクセス制御をAzure ADへ移行した後は、認証を切り替えます。

#### 認証切り替えに際した考慮ポイント
パスワードハッシュ同期を有効化し、認証をAzure ADにて実施するよう切り替えた場合、以下のような制限事項があります。マイクロソフトは、これらを解消するための機能追加を検討中です。これらの制限事項が解消されるまでAD FS撤廃は難しいかもしれませんが、上記したその他の機能移行はメリットをもたらしますので、準備を進めておくことを強くお奨めいたします。
* オンプレミスのADでパスワード失効後でも、Azure ADのアカウントでサインイン可能
* オンプレミスのADでアカウントロックアウト後でも、Azure ADのアカウントでサインイン可能
* オンプレミスのADでアカウント無効化後、それがAzure ADへ反映されるまで最大30分(同期サイクルの規定値)かかる
* レガシー認証のブロックが限定的：Outlook2010等の古いOfficeクライアントからの認証をブロックするには、Exchange側でコントロール（社内/社外とわずブロック等）する必要がある
* スマートカード認証等のクライアント証明書を使った認証はAD FSのみでサポート  
※その他は、[ハイブリッドID認証方式の比較](HybridId-Comparison.md)を参照ください。
