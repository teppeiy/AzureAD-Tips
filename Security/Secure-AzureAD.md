# Azure AD・認証基盤のセキュリティ対策

## 概要
昨今、増大しているセキュリティインシデントの頻度や影響度は、ITの課題ではなく会社の存続をも脅かす経営課題です。75％のセキュリティインシデントは、IDが盗まれたことに起因しているというデータもあり、IDを守ることは非常に有効な防御策です。にもかかわらず、まだ対策が万全でない企業も多いのが現状です。
Azure AD等、クラウド認証基盤で必ずすべきセキュリティ対策を記載します。
## 推奨事項
* [特権アカウントの保護](#特権アカウントの保護)
    * 多要素認証の有効化
    * Just-In-Time権限昇格機能(PIM)の有効化
* [リスクのフラグ付きユーザーのレポートの有効化](#リスクのフラグ付きユーザーのレポートの有効化)
* [パスワードポシリーの強化](#パスワードポシリーの強化)
* [ADFSの保護 (ADFS利用の場合)](#ADFSの保護)
    * ADFS Extranet Lockout Protection の有効化
    * Azure AD Connect Health for ADFS の有効化


## 特権アカウントの保護
特権アカウント、特に全体管理者 (Global Admin) や、Exchange管理者権限が盗まれた場合には、会社を乗っ取られたと言っても過言ではないぐらいの影響を及ぼすため、**必ず**特別な保護してください。
* ### [多要素認証(MFA)](https://docs.microsoft.com/ja-jp/azure/multi-factor-authentication/multi-factor-authentication)を強制
    特権アカウントグループを作成し、[条件付きアクセス](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-conditional-access-azure-portal)を利用してMFAを強制します。  
    もしくは、個々の特権アカウントに対し、MFAを強制します。
* ### Just-In-Time 権限昇格機能 (PIM) の有効化  
    時間制限付の権限昇格機能である[Privileged Identity Protection (PIM)](https://docs.microsoft.com/ja-jp/azure/active-directory/privileged-identity-management/active-directory-securing-privileged-access) は、最低限必要な時間帯のみ権限の行使を許可することで、攻撃からのリスクを劇的に軽減することができます。Azure AD Premium P2ライセンスが必要ですが、特権アカウント分だけでも購入する価値はあります。  
* ### 特権アクセス ワークステーション (PAW) の利用  
    更に強固な保護には、専用マシンからのみのアクセスを許可するよう実装をしてください。[条件付きアクセス](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-conditional-access-azure-portal)や、ADFSのIPアドレス制限、証明書認証等を利用することで実装します。
    詳しくは[こちらのドキュメント](https://docs.microsoft.com/ja-jp/windows-server/identity/securing-privileged-access/privileged-access-workstations)を参照ください。  

#### Breaking Glass シナリオ
WIP

## [リスクのフラグ付きユーザーのレポート](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-identityprotection#users-flagged-for-risk) の有効化
マイクロソフトは、ブラックマーケット等の複数のソースから、漏洩した資格情報（ID/パスワード）一覧を継続的に取得しています。そのリストとAzure ADのアカウントと機械的に突き合わせることにより、侵害された可能性の高いユーザーに関するレポートを提供しています。  
この機能を利用するには、[パスワードハッシュ同期](https://docs.microsoft.com/ja-jp/azure/active-directory/connect/active-directory-aadconnectsync-implement-password-synchronization)が必須です。


### パスワードハッシュをAzure ADに同期することの安全性について
パスワードハッシュと言えども、Azure ADに同期することに懸念を持つ方々も少なくありません。その場合、同期プロセスや[Smart Password Lockout](https://docs.Microsoft.com/ja-jp/azure/active-directory/active-directory-secure-passwords#azure-ad-password-protections)の説明が助けになるかもしれません。  

#### パスワードハッシュ同期のメカニズム
オンプレミスActive Directoryには、MD4でハッシュ化されたパスワードが保存されています。パスワードハッシュ同期を有効化すると、Azure AD Connectは、元々のパスワードハッシュをソルトやストレッチングといったテクニックを使い、攻撃リスクを極限まで抑えるプロセスを経てAzure ADへ同期します。
* ソルト: ハッシュ化前に対象文字列にランダムな文字列を不可する。レインボーテーブルによる探索に対する防御策の一つ
* ストレッチング: 何度もハッシュ化をすることにより、攻撃者の解析コストを高める（時間の長期化）防御策の一つ

具体的には、ADに保存されているMD4でハッシュ化されたパスワードにsaltを付加、[PBKDF2](https://www.ietf.org/rfc/rfc2898.txt)関数で計算後、[HMAC-SHA256](https://msdn.microsoft.com/library/system.security.cryptography.hmacsha256.aspx)
を使ってキー付きハッシュアルゴリズムで1,000回ものハッシュ化を実施する、といったような一連のプロセスです。詳しくは[こちらのドキュメント](https://docs.microsoft.com/ja-jp/azure/active-directory/connect/active-directory-aadconnectsync-implement-password-synchronization#how-password-synchronization-works)を参照ください。  

#### [Smart Password Lockout](https://docs.Microsoft.com/ja-jp/azure/active-directory/active-directory-secure-passwords#azure-ad-password-protections)
Azure ADの基本機能である[Smart Password Lockout](https://docs.Microsoft.com/ja-jp/azure/active-directory/active-directory-secure-passwords#azure-ad-password-protections)は、ユーザーのパスワードがサイバー犯罪者によってハッキングされようとしている可能性があることを Azure AD が検出すると、そのユーザー アカウントをロックします。 Azure AD は、特定のログイン セッションについてのリスクを判別できるように設計されています。 サイバー脅威を止めるために、最新のセキュリティ データを利用してロックアウト セマンティクスを適用します。

### 手順
1. [Azure AD Connectの構成ウィザードを利用し、パスワードハッシュ同期を有効化する](https://docs.microsoft.com/ja-jp/azure/active-directory/connect/active-directory-aadconnectsync-implement-password-synchronization#enable-password-synchronization)
2. [リスクのフラグ付きユーザーのレポートを確認する](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-reporting-security-user-at-risk)

### パスワードハッシュ同期をすることによるその他のメリット  
#### 1. ディザスタリカバリへの対策  
フェデレーション環境において、ADFS/WAPが利用できなくなった際には、Office 365等Azure ADに認証を依存しているサービスへのログインが利用できなくなります。実際にADFSがPetya等のランサムウェアに感染した企業において、パスワードハッシュ同期をしていた企業はADFSからAzure ADへの認証に切り替えることによって、数時間のダウンタイムでサービスへの認証アクセスが復旧できました。一方、パスワードハッシュ同期をしていなかったため、ADFSを構築しなおす等の作業で、数日間のダウンタイムを強いられた企業もあります。
#### 2. 近い将来ADFSを廃止することへの準備  
これまで多くのケースでADFSが必要でしたが、昨今のAzure ADの急速な進化により、ADFSが無くとも多くの要件が満たせるようになってきました。その例として、[条件付きアクセス](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-conditional-access-azure-portal)を利用することにより、アクセス元IPアドレスに基づいたアクセス許可・拒否が可能になってきています。また、[シームレスシングルサインオン(SeamlessSSO)](https://docs.microsoft.com/ja-jp/azure/active-directory/connect/active-directory-aadconnect-sso)を利用することで、シングルサインオンも実現可能になりました。  
ADFSの廃止については、[こちら](Goodbye-ADFS.md)も参照ください。

## パスワードポシリーの強化
* ### [動的な禁止パスワードポリシー](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-secure-passwords)の有効化   
    Azure AD と Microsoft アカウントでは、パスワードを確実に保護するために、よく使われているパスワードが動的に禁止されています。 Azure AD Identity Protection チームは、禁止パスワード リストを定期的に分析し、ありきたりのパスワードをユーザーが選択できないようにしています。 このサービスは、Azure AD と Microsoft アカウント サービスのユーザーが利用できます。
    この機能は、[セルフサービスパスワードリセット](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-passwords-overview)を有効化することにより利用可能になります。
## ADFSの保護
* ### [ADFS Extranet Lockout Protection](https://docs.microsoft.com/ja-jp/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-lockout-protection) の有効化  
    ADFS利用の場合、この機能を有効化することにより、ADFSへのブルートフォースアタック等への対策になります。  
* ### [Azure AD Connect Health for ADFS](https://docs.microsoft.com/ja-jp/azure/active-directory/connect-health/active-directory-aadconnect-health) の有効化  
    この機能を有効化することで、無効なユーザー名とパスワードによる試行を行った上位 50 人のユーザーと直近の IP アドレスなど、ADFS に関する[レポート](https://docs.microsoft.com/ja-jp/azure/active-directory/connect-health/active-directory-aadconnect-health-adfs)を作成することができます。