# ADFS廃止のすすめ

## 概要
これまで、Office 365等へのシングルサインオンやアクセス制御を AD FS で実施するというケースが主流でしたが、最近ではAzure ADへのシフトが加速化されています。AD FS を利用せず Azure AD での認証に切り替えるメリットとして以下のような点が挙げられます。

* ### **世界トップクラスのセキュリティのメリットを享受**  
  AD FS に対する攻撃対策は、それを運用している企業の責任範囲ですが、それをマイクロソフトへ任せることで、インテリジェントなセキュリティ対策のメリットを享受することができます。AD FS を運用すると企業ネットワーク（実際にはDMZ）への受信ポート 443 を開放することになり、日々進化するあらゆる攻撃への対策は非常に困難でコストもかかります。  
  AD FS を無くすことで、アタックサーフェスもなくなり、また、Azure AD の基本機能である [Smart Lockout](https://docs.microsoft.com/ja-jp/azure/active-directory/authentication/howto-password-smart-lockout) を利用することになり、パスワードスプレー等の様々な攻撃から自動的に防御されます。
* ### [**条件付きアクセス**](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-conditional-access-azure-portal)**で、より高度できめ細かなアクセス制御を実現**  
  複雑な構文を要する AD FS のクレームルールを管理することは手間がかかりますし、変更管理やテストにかかる労力や変更適用までの時間は小さくありません。条件付きアクセスを利用することで、GUI を使った形でのアクセスポリシーを管理できます。また、AD FS では単一アプリケーションとして扱わざるを得なかった Office 365 アプリケーションですが、条件付きアクセスでは各アプリケーション毎に独立したルールを定義することが可能です。
* ### **オンプレミス環境の管理から解放**  
  AD FSを利用する場合、冗長性や災害対策を考慮すると最低 8 台（(AD FS 2 台 + WAP 2 台)* 2 サイト）また、テスト・QA 環境も含めると更に多くのサーバーの管理が必要です。加えて、各種証明書の管理、DMZ を含む外部からのアクセスセキュリティの管理、ロードバランサーの管理、セキュリティ更新プログラムの管理等、比較的大きなコストと労力が必要ですが、これらから開放されます。

参考：メリットや事例についてはこちらのビデオもご覧ください  
[【de:code 2018】CI03 AD FS では守れない？！アカウント乗っ取りを防ぐためにすべき 3 つのこと ～ユーザー企業の実例のご紹介～](https://www.youtube.com/watch?v=g2mB_EKqi-g)

## AD FS 廃止までのステップ
1. 準備  
    1.1. パスワードハッシュ同期の有効化（認証の切り替え無し）  
    1.2. AD FSに依存しているサービスを移行  
    * アプリケーション（証明書利用者信頼）
    * アクセス制御 (クレームルール等)
    * デバイス登録サービス
2. 認証の切り替え
3. 廃止

## 1.1. パスワードハッシュ同期の有効化（認証の切り替え無し）
既存のフェデレーション（実際に認証する場所は AD FS）設定に変更を加えず、オンプレミス AD のパスワードハッシュのハッシュをAzure ADへ同期します。繰り返しになりますが、**この時点で認証フローが変更されることはありません。つまり、これまで通り AD FS を利用した認証が維持されます。**  
パスワードハッシュ同期は、AD FS を無くすためのステップの一つですが、2つの大きなメリットがあります。
1. [漏洩した資格情報検知レポート](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-identityprotection#users-flagged-for-risk) によるセキュリティ向上  
マイクロソフトは、複数のソースから漏洩した資格情報一覧を継続的に取得しています。そのリストと Azure AD 利用者のアカウントを機械的に突き合わせることにより、漏洩した資格情報を検知しレポートを提供します。企業の管理者は、漏洩しているアカウントを知ることができ、被害を拡大させないような対策（パスワードをリセットする等）を実施できます。
2. ディザスタリカバリの対策  
フェデレーション環境において、AD FS/WAP が利用できなくなった際には、Office 365 等 Azure AD に認証を依存しているサービスが利用できなくなります。実際に AD FS が Pety aに感染した企業において、パスワードハッシュ同期をしていた企業は AD FS から Azure AD への認証に切り替えることによって、数時間のダウンタイムでサービスへの認証アクセスが復旧できました。一方、パスワードハッシュ同期をしていなかったため、AD FS を構築しなおす等の作業で、数日間のダウンタイムを強いられた企業もあります。

#### パスワードハッシュを Azure AD に同期することの安全性について
パスワードハッシュと言えども、Azure AD に資格情報を同期することに懸念を持つ方々(情報セキュリティ部門等)も少なくありません。その場合、同期プロセスや[Smart Lockout](https://docs.microsoft.com/ja-jp/azure/active-directory/authentication/howto-password-smart-lockout) の説明が助けになるかもしれません。  

#### パスワードハッシュ同期のメカニズム
パスワードハッシュ同期を有効化すると、Azure AD Connectは、元々のパスワードハッシュをソルト + ストレッチング + ハッシュといったテクニックを使い、攻撃リスクを軽減するプロセスを通してAzure ADへ同期します。
* ソルト: ハッシュ化前に対象文字列にランダムな文字列を不可する。レインボーテーブルによる探索に対する防御策の一つ
* ストレッチング: 何度もハッシュ化をすることにより、攻撃者の解析コストを高める（時間の長期化）防御策の一つ


    参考：
    [ハッシュとソルト、ストレッチングを正しく理解する：本当は怖いパスワードの話](http://www.atmarkit.co.jp/ait/articles/1110/06/news154.html)


具体的には、ADに保存されているMD4でハッシュ化されたパスワードにソルトを付加、[PBKDF2](https://www.ietf.org/rfc/rfc2898.txt)関数で計算後、[HMAC-SHA256](https://msdn.microsoft.com/library/system.security.cryptography.hmacsha256.aspx)
を使ってキー付きハッシュアルゴリズムで1,000回ものハッシュ化を実施する、といったような一連のプロセスです。詳しくは[こちらのドキュメント](https://docs.microsoft.com/ja-jp/azure/active-directory/connect/active-directory-aadconnectsync-implement-password-synchronization#how-password-synchronization-works)を参照ください。  

#### 手順:
1. [Azure AD Connect の構成ウィザードを利用し、パスワードハッシュ同期を有効化する](https://docs.microsoft.com/ja-jp/azure/active-directory/hybrid/plan-migrate-adfs-password-hash-sync#implementing-your-solution)
2. [漏洩した資格情報検知レポートを確認する](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-reporting-security-user-at-risk)

#### Azure AD に対する攻撃への保護
* [Smart Lockout](https://docs.microsoft.com/ja-jp/azure/active-directory/authentication/howto-password-smart-lockout)  
ユーザーのパスワードがサイバー犯罪者によってハッキングされようとしている可能性があることを Azure AD が検出すると、[Smart Lockout](https://docs.microsoft.com/ja-jp/azure/active-directory/authentication/howto-password-smart-lockout) 機能が働くことで、そのユーザー アカウントをロックします。

#### 更にセキュリティレベルを上げる
* [Identify Protection](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-identityprotection) (Azure AD Premium P2 の機能) による自動応答機能を利用することで、たとえば、漏洩した資格情報が見つかると、そのアカウントを利用してログインをブロックしたり、パスワード変更を強制するといったような自動対応が可能になります。


参考:  
Ignite 2017 - [Shut the door to cybercrime with Azure Active Directory risk-based identity protection](https://myignite.microsoft.com/sessions/53404?source=sessions)

## 1.2. AD FS に依存しているサービスを移行

### 移行対象の代表的なサービス
* アプリケーション (証明書利用者信頼)  
AD FS を認証プロバイダー(IdP)としているアプリケーションです。多くの場合、AD FS の機能の移行対象となる Azure AD そのものと、それ以外のアプリケーションを分けて考えて移行戦略を立てます。
* アクセス制御
これは上記したアプリケーションと一緒に考慮する必要があります。発行承認ルール (Issuance Authorization Rules) 、いわゆるクレームルールを Azure AD の条件付きアクセスへ移行していきます。
* デバイス登録サービス (DRS)  
この機能が利用されているケースはとても少ないですが、AD FS のデバイス登録サービスを利用している場合には、Azure AD DRSへ移行します。

### アプリケーション (証明書利用者信頼)
AD FS を認証プロバイダー(IdP)としているアプリケーションを棚卸しします。そして、証明書利用者信頼毎の発行変換ルール (Issuance Transformation Rules) が Azure AD で実装可能か確認します。  
詳しくは、[アプリケーションを AD FS から Azure AD に移行するためのガイダンス](https://docs.microsoft.com/ja-jp/azure/active-directory/manage-apps/migrate-adfs-apps-to-azure) を参考にします。

### アクセス制御
承認要求規則から[条件付きアクセス](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-conditional-access-azure-portal)へ移行します。既存のルールをそのまま移行するのではなく、時代背景に合った今後のアクセス制御ポリシーを再検討することを強くお奨めします。  
以下をご参考ください。

* Azure AD ウェビナー：[IP ベースのアクセス制御から脱却してよりセキュアな環境を構築しよう](http://aka.ms/ztnwebinar) 
* [Microsoft 365 の推奨ポリシー](
https://docs.microsoft.com/ja-jp/microsoft-365/enterprise/microsoft-365-policies-configurations)
* [Azure AD 条件付きアクセスの FAQ](https://github.com/teppeiy/AzureAD-Tips/blob/master/CA/CA-Faq.md)

#### 考慮ポイント
* レガシープロトコルの条件付きアクセス対応 → 対応済み  
~~現状、Office2010等、レガシープロトコルを利用するトラフィックは、Azure ADの条件付きアクセスの対象外となり、AD FSでしか制御することができません。メールプロトコル(POP/IMAP/ActiveSync)については、Exchange Onlineのポリシーで有効化・無効化することが可能です。~~
* ライセンス  
AD FSでクレームルールを利用し、場所ベースのアクセス制御をしている利用ケースは少なくありません。これと同様あるいは更に高度な機能な有する条件付きアクセスを利用することになりますが、Azure AD Premium P1 ライセンスが必要になります。これまで、ITシステムの利用が限られているユーザー（例：工場勤務の従業員等）がライセンスを保持していない場合には、追加購入の検討が必要になります。詳しくは、マイクロソフトの営業担当者へご相談ください。
* 証明書を利用した端末特定をしている場合  
会社支給端末に証明書を配布し、AD FS でユーザーID/パスワード に加え、証明書の存在を確認してアクセス許可する、という利用ケースを多く見ます。その場合には、更にセキュアで管理コストが低い代替ソリューションで同様な要件を満たすことができる場合が多いです。  
具体的には、Intune のポリシーが適用されている会社支給端末のみアクセスを許可、あるいは、ドメイン参加した端末のみアクセスを許可、という制御をすることで代替します。[こちら](https://github.com/teppeiy/AzureAD-Tips/blob/master/Security/Device-Posture.md)も参考ください。


#### アクセス制御の移行戦略
既存ユーザーへの影響を最小化しながら、効率的にアクセス制御を移行する方法について紹介します。  
* 準備
  * オンプレミスのセキュリティグループ（Migration Group とします）を作成し、それを Azure AD へ同期しておきます。
  * Migration Group は、AD FS のクレームルールをバイパスするように設定し、AD FS によるアクセス制御の影響を受けないようにしておきます。
    * 既にクレームルールをバイパスされるグループが存在している場合は Migration Group をそのグループのメンバーとして入れ子にすることで、既存の AD FS のクレームルールを変更する必要がなくなります。
  * 条件付きアクセスの対象として、Migration Group を選択しておきます。
* テスト・移行
  * Migration Group へ一部のユーザーを追加し、条件付きアクセスポリシーで意図されたアクセス制御が実現できているか検証します
  * 徐々に Migration Group のメンバーユーザーを増やし、影響を確認しながら移行を進めていきます。
  * 最終的には、移行対象としたい全てのユーザーを Migration Group のメンバーとすることで、移行を完了させます。
##### この移行方式の問題点
オンプレミス AD で Migration Group に対象ユーザーを追加し、Azure AD Connect によってグループメンバーシップ情報が Azure AD へ同期されるまでの間（最大30分程度、同期周期に依存）、AD FS のクレームルールはバイパス、条件付きアクセスの対象にならない（＝アクセス許可）という状態が存在します。  
リスク軽減方法としては、Migration Group にメンバーを追加した直後に同期をする等の対策が考えられます。

### デバイス登録サービス (DRS)  
AD FS デバイス登録サービスを代替する Azure AD 側の実装方式は複数存在しています。自組織で用いるデバイス種別 (PC, モバイル, 各々の OS 種別バージョンなど) を鑑み、最も適合した [Azure AD 上のデバイス管理方式を選択](https://docs.microsoft.com/ja-jp/azure/active-directory/device-management-introduction)し、デバイス管理のインフラ側設定を実施します。

## 2. 認証の切り替え
パスワードハッシュ同期の有効化、AD FSに依存しているアプリ(証明書利用者信頼)やアクセス制御を Azure AD へ移行した後は、フェデレーションが構成されたドメインの認証方式の切り替え作業を行います。

#### 認証切り替えに際した考慮ポイント
パスワードハッシュ同期を有効化し、認証をAzure ADにて実施するよう切り替えた場合、以下のような制限事項があります。マイクロソフトは、これらを解消するための機能追加を検討中です。これらの制限事項が解消されるまでAD FS撤廃が難しいケースもあるかもしれませんが、前述した機能の移行は事前準備として実施可能であり、沢山のメリットをもたらすため実施を強く推奨します。
* パスワードの期限切れについて - 問題になることは殆どありません。詳しくは、[こちら](/Hybrid/Password-Expiration.md)をご参考ください。
  * オンプレミスのADでパスワード失効後でも、Azure ADのアカウントでサインイン可能。
  * パスワード期限切れが Win10 や Office ポータル上に通知されない。[代替ソリューション](https://gallery.technet.microsoft.com/scriptcenter/Password-Expiry-Email-177c3e27)を検討します。
* オンプレミスの AD でアカウント無効化後、それが Azure AD へ反映されるまで最大30分(同期サイクルの規定値)かかる。
* オンプレミスの AD でアカウントロックアウト後でも、Azure ADのアカウントでサインイン可能。
* * スマートカード認証等のクライアント証明書を使った認証はAD FSのみでサポート。
* ~~レガシー認証のブロックが限定的：Outlook2010等の古いOfficeクライアントからの認証をブロックするには、Exchange側でコントロール（社内/社外とわずブロック等）する必要がある~~
* その他は、[ハイブリッドID認証方式の比較](/Hybrid/HybridId-Comparison.md)を参照ください。

#### 手順
[AD FS からパスワードハッシュ同期認証(PHS)への移行ガイド](https://docs.microsoft.com/ja-jp/azure/active-directory/hybrid/plan-migrate-adfs-password-hash-sync)   
[AD FS からパススルー認証(PTA)への移行ガイド](https://docs.microsoft.com/ja-jp/azure/active-directory/hybrid/plan-migrate-adfs-pass-through-authentication)

## 3. 廃止
AD FS に依存しているサービスが無いことを確認し、電源を切ります。
