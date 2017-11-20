# Azure AD・認証基盤のセキュリティ対策

## 概要
昨今、増大しているセキュリティインシデントの頻度や影響度は、ITの課題ではなく会社の存続をも脅かす経営課題です。75％のセキュリティインシデントは、IDが盗まれたことに起因しているというデータもあり、IDを守ることは非常に有効な防御策です。にもかかわらず、まだ対策が万全でない企業も多いのが現状です。
Azure AD等、クラウド認証基盤で必ずすべきセキュリティ対策を記載します。
## 推奨事項
* [特権アカウントの保護](#特権アカウントの保護)
    * 多要素認証(MFA) の有効化
    * Privileged Identity Management(PIM) の有効化
* [漏洩したIDのレポートの有効化](#漏洩したIDのレポートの有効化)
    * パスワードハッシュ同期の有効化
* [パスワードポシリーの強化](#パスワードポシリーの強化)
    * セルフサービスパスワードリセットの有効化
* [AD FSの保護](#ad-fsの保護)
    * AD FS Extranet Lockout Protection の有効化
    * Azure AD Connect Health for AD FS の有効化


## 特権アカウントの保護
特権アカウント、特に全体管理者 (Global Admin) や、Exchange管理者権限が盗まれた場合には、会社を乗っ取られたと言っても過言ではないぐらいの影響を及ぼすため、**必ず**特別な保護してください。
* ### 全体管理者数は極力少なく
    ほとんどの企業では、全体管理者は多くても2～3アカウントで十分に事足りるはずです。必要な管理作業を遂行することができる最小の権限を[管理者ロール](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-assign-admin-roles-azure-portal)の中から割り当てることで、特権アカウントの意図しない利用や不正な利用のリスクを最小限にします。
    <!--#### Breaking Glass シナリオWIP-->

* ### [多要素認証(MFA)](https://docs.microsoft.com/ja-jp/azure/multi-factor-authentication/multi-factor-authentication)を強制
    特権アカウントグループを作成し、[条件付きアクセス](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-conditional-access-azure-portal)を利用してMFAを強制します。  
    もしくは、個々の特権アカウントに対し、MFAを強制します。
* ### Just-In-Time 権限昇格機能 (PIM) の有効化  
    時間制限付の権限昇格機能である[Privileged Identity Protection (PIM)](https://docs.microsoft.com/ja-jp/azure/active-directory/privileged-identity-management/active-directory-securing-privileged-access) は、最低限必要な時間帯のみ権限の行使を許可することで、攻撃からのリスクを劇的に軽減することができます。Azure AD Premium P2ライセンスが必要ですが、特権アカウント分だけでも購入する価値はあります。  
    PIMを有効化した場合でも、普段メール等で利用するアカウントと特権アカウントを分けて利用することをお奨めしています。
* ### 特権アクセス ワークステーション (PAW) の利用 (オプション)  
    更に強固な保護には、専用マシンからのみのアクセスを許可するよう実装をしてください。[条件付きアクセス](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-conditional-access-azure-portal)や、AD FSのIPアドレス制限、証明書認証等を利用することで実装します。
    詳しくは[こちらのドキュメント](https://docs.microsoft.com/ja-jp/windows-server/identity/securing-privileged-access/privileged-access-workstations)を参照ください。  


## [漏洩したIDのレポートの有効化](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-identityprotection#users-flagged-for-risk) の有効化
マイクロソフトは、ブラックマーケット等の複数のソースから、漏洩した資格情報（ID/パスワード）一覧を継続的に取得しています。そのリストとAzure ADのアカウントと機械的に突き合わせることにより、漏洩したアカウントに関するレポートを提供しています。  
この機能を利用するには、[パスワードハッシュ同期](https://docs.microsoft.com/ja-jp/azure/active-directory/connect/active-directory-aadconnectsync-implement-password-synchronization)をするだけです。  
AD FS等を利用するフェデレーション環境においても、パスワードハッシュ同期を強くお奨めしています。フェデレーション環境においてパスワードハッシュ同期を有効にしても、ログインフローへの影響は一切ありません。


### パスワードハッシュをAzure ADに同期することの安全性について
パスワードハッシュと言えども、Azure ADに資格情報を同期することに懸念を持つ方々も少なくありません。その場合、同期プロセスや[Smart Lockout](https://docs.Microsoft.com/ja-jp/azure/active-directory/active-directory-secure-passwords#azure-ad-password-protections)の説明が助けになるかもしれません。  

#### パスワードハッシュ同期のメカニズム
パスワードハッシュ同期を有効化すると、Azure AD Connectは、元々のパスワードハッシュをソルト+ストレッチング+ハッシュといったテクニックを使い、攻撃リスクを軽減するプロセスを通してAzure ADへ同期します。
* ソルト: ハッシュ化前に対象文字列にランダムな文字列を不可する。レインボーテーブルによる探索に対する防御策の一つ
* ストレッチング: 何度もハッシュ化をすることにより、攻撃者の解析コストを高める（時間の長期化）防御策の一つ


    参考：
    [ハッシュとソルト、ストレッチングを正しく理解する：本当は怖いパスワードの話](http://www.atmarkit.co.jp/ait/articles/1110/06/news154.html)


具体的には、ADに保存されているMD4でハッシュ化されたパスワードにsaltを付加、[PBKDF2](https://www.ietf.org/rfc/rfc2898.txt)関数で計算後、[HMAC-SHA256](https://msdn.microsoft.com/library/system.security.cryptography.hmacsha256.aspx)
を使ってキー付きハッシュアルゴリズムで1,000回ものハッシュ化を実施する、といったような一連のプロセスです。詳しくは[こちらのドキュメント](https://docs.microsoft.com/ja-jp/azure/active-directory/connect/active-directory-aadconnectsync-implement-password-synchronization#how-password-synchronization-works)を参照ください。  

#### [Smart Lockout (非フェデレーション環境において)](https://docs.Microsoft.com/ja-jp/azure/active-directory/active-directory-secure-passwords#azure-ad-password-protections)
Azure ADの基本機能である[Smart Lockout](https://docs.Microsoft.com/ja-jp/azure/active-directory/active-directory-secure-passwords#azure-ad-password-protections)は、ブルートフォース攻撃等への防御策です。
既定のロックアウトしきい値は試行失敗 10 回で、既定のロックアウト期間は 60 秒です。
また、スマート ロックアウトは正規のユーザーによるサインインと攻撃者によるサインインを区別し、ほとんどの場合は攻撃者のみをロックアウトします。 この機能は、攻撃者の悪意によって正規のユーザーがロックアウトされるのを防ぎます。 正規のユーザーと攻撃者を区別するには、過去のサインイン動作、ユーザーのデバイスとブラウザー、その他のシグナルが使われます。 アルゴリズムは常に改善されています。

フェデレーション環境においては、同様な防御策をAD FS等で行う必要がありますが、それについては [AD FSの保護](#ad-fsの保護) をご覧ください。

### 手順
1. [Azure AD Connectの構成ウィザードを利用し、パスワードハッシュ同期を有効化する](https://docs.microsoft.com/ja-jp/azure/active-directory/connect/active-directory-aadconnectsync-implement-password-synchronization#enable-password-synchronization)
2. [リスクのフラグ付きユーザーのレポートを確認する](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-reporting-security-user-at-risk)

### 自動対処
[Azure AD Identity Protection](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-identityprotection)を利用すると、漏洩したアカウントを検出すると、次回ログイン時にMFAやパスワード変更を強制することが可能です。[Azure AD Identity Protection](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-identityprotection)を利用するにはAzure AD Premium P2ライセンスが必要になります。

### パスワードハッシュ同期の有効化によるその他のメリット  
#### 1. ディザスタリカバリへの対策  
フェデレーション環境において、AD FS/WAPが利用できなくなった際には、Office 365等Azure ADに認証を依存しているサービスへのログインが利用できなくなります。実際にAD FSがPetya等のランサムウェアに感染した企業において、パスワードハッシュ同期をしていた企業はAD FSからAzure ADへの認証に切り替えることによって、数時間のダウンタイムでサービスへの認証アクセスが復旧できました。一方、パスワードハッシュ同期をしていなかったため、AD FSを構築しなおす等の作業で、数日間のダウンタイムを強いられた企業もあります。
#### 2. 近い将来AD FSを廃止することへの準備  
これまで多くのケースでAD FSが必要でしたが、昨今のAzure ADの急速な進化により、AD FSが無くとも多くの要件が満たせるようになってきました。その例として、[条件付きアクセス](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-conditional-access-azure-portal)を利用することにより、アクセス元IPアドレスに基づいたアクセス許可・拒否が可能になってきています。また、[シームレスシングルサインオン(SeamlessSSO)](https://docs.microsoft.com/ja-jp/azure/active-directory/connect/active-directory-aadconnect-sso)を利用することで、シングルサインオンも実現可能になりました。  
AD FSの廃止については、[こちら](Goodbye-ADFS.md)も参照ください。

## パスワードポシリーの強化
* ### [脆弱なパスワード利用禁止ポリシー](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-secure-passwords)の有効化   
    Azure AD と Microsoft アカウントでは、よく攻撃に利用されているパスワードを保持しています。そして、それらのパスワードをユーザーが設定できないようなメカニズムを持っています。
    この機能は、[セルフサービスパスワードリセット](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-passwords-overview)を有効化することにより利用可能になります。  
    注；同期IDを利用している場合、ユーザーが直接オンプレミスのADでパスワードを更新した場合には、よく利用される脆弱なパスワード禁止ポリシーは適用されません。今後の機能拡張において、オンプレミスのADでも禁止ポリシーが適用できるような仕組みを検討中です。
## AD FSの保護
* ### [AD FS Extranet Lockout Protection](https://docs.microsoft.com/ja-jp/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-lockout-protection) の有効化  
    AD FS利用の場合、この機能を有効化することにより、AD FSへのブルートフォース攻撃等への対策になります。  

* ### [Azure AD Connect Health for AD FS](https://docs.microsoft.com/ja-jp/azure/active-directory/connect-health/active-directory-aadconnect-health) の有効化  
    この機能を有効化することで、無効なユーザー名とパスワードによる試行を行った上位 50 人のユーザーと直近の IP アドレスなど、AD FS に関する[レポート](https://docs.microsoft.com/ja-jp/azure/active-directory/connect-health/active-directory-aadconnect-health-ADFS)を作成することができます。

<!--
* ### [AD FSのプライマリ認証にワンタイムパスコードを利用する (オプション)](https://docs.microsoft.com/ja-jp/windows-server/identity/ad-fs/operations/configure-ad-fs-and-azure-mfa)
    Windows Server 2016ベースのAD FSとAzure MFAの組み合わせで、プライマリ認証にワンタイムパスコードを利用することが可能です。ブルートフォース攻撃やパスワードスプレー攻撃等への対策になります。

    [![Azure MFA as AD FS Primary Authentication Method](http://img.youtube.com/vi/vlEE5DqpwUs/0.jpg)](http://www.youtube.com/watch?v=vlEE5DqpwUs)

-->