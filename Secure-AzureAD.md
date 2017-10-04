# Azure AD・認証基盤の保護

## 概要
昨今、増大しているセキュリティインシデントの頻度や影響度は、ITの課題ではなく会社の存続をも脅かす経営課題です。75％のセキュリティインシデントは、IDが盗まれたことで発生しているというデータもあり、IDを守ることは非常に有効な防御策であるにもかかわらず、まだ対策が万全でない企業も多いのが現状です。
Azure AD等、クラウド認証基盤で必ずすべきセキュリティ対策を記載します。
## 推奨事項
* [特権アカウントの保護](#特権アカウントの保護)
    * 多要素認証の有効化
    * JIT昇格機能の有効化
* [資格情報漏洩検知レポートの有効化](#資格情報漏洩検知レポートの有効化)
* [パスワードポシリーの強化](#パスワードポシリーの強化)
* [ADFSの保護](#ADFSの保護)
    * ブルートフォースアタック保護機能の有効化
    * 監査レポート（Azure AD Connect Health）の有効化


## 特権アカウントの保護
特権アカウント、特に全体管理者（Global Admin）や、Exchange管理者権限が盗まれた場合には、会社を乗っ取られたと言っても過言ではないぐらいの影響を及ぼすため、必ず特別な保護してください。
* ### 多要素認証（MFA）を強制
    特権アカウントグループを作成し、[条件付きアクセス](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-conditional-access-azure-portal)を利用してMFAを強制します。  
    もしくは、個々の特権アカウントに対し、MFAを強制します。
* ### Privileged Identity Protection (PIM) の有効化  
    時間制限付の権限昇格機能である[PIM](https://docs.microsoft.com/ja-jp/azure/active-directory/privileged-identity-management/active-directory-securing-privileged-access)は、最低限必要な時間帯のみ権限の行使を許可することで、攻撃からのリスクを劇的に軽減することができます。Azure AD Premium P2ライセンスが必要ですが、特権アカウント分だけでも購入する価値はあります。  
* ### Secure Admin Workstation (SAW) の利用  
    更に強固な保護をするには、専用マシンからのみのアクセスを許可するよう実装をしてください。[条件付きアクセス](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-conditional-access-azure-portal)や、ADFSのIPアドレス制限、証明書認証等を利用することで実装します。
#### Breaking Glass シナリオ
xxxx

## 資格情報漏洩検知レポートの有効化
### パスワードハッシュ同期の有効化
パスワードハッシュ同期は、2つのメリットをもたらします。
1. 漏洩した資格情報検知レポートによるセキュリティ向上  
マイクロソフトは、複数のソースから漏洩した資格情報一覧を継続的に取得しています。そのリストとAzure ADアカウントと機械的に突き合わせることにより、漏洩した資格情報を検知しレポートを提供しています。
2. ディザスタリカバリへの対策  
フェデレーション環境において、ADFS/WAPが利用できなくなった際には、Office 365等Azure ADに認証を依存しているサービスが利用できなくなります。実際にADFSがPetyaに感染した企業において、パスワードハッシュ同期をしていた企業はADFSからAzure ADへの認証に切り替えることによって、数時間のダウンタイムでサービスへの認証アクセスが復旧できました。一方、パスワードハッシュ同期をしていなかったため、ADFSを構築しなおす等の作業で、数日間のダウンタイムを強いられた企業もあります。

#### パスワードハッシュをAzure ADに同期することの安全性について
オンプレミスActive Directoryには、MD4でハッシュ化されたパスワードが保存されています。パスワードハッシュ同期を有効化すると、Azure AD Connectでは、salt + PBKDF2 + HMAC - SHA256 のプロセスでハッシュ化した上でAzure ADへ保存されます。  
https://docs.microsoft.com/ja-jp/azure/active-directory/connect/active-directory-aadconnectsync-implement-password-synchronization#how-password-synchronization-works

#### ステップ:
1. Azure AD Connectの構成ウィザードを利用し、パスワードハッシュ同期を有効化する  
https://docs.microsoft.com/ja-jp/azure/active-directory/connect/active-directory-aadconnectsync-implement-password-synchronization#enable-password-synchronization
2. 漏洩した資格情報検知レポートを確認する  
https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-reporting-security-user-at-risk

#### Azure AD のパスワード保護
Smart Lockout  
ユーザーのパスワードがサイバー犯罪者によってハッキングされようとしている可能性があることを Azure AD が検出すると、Microsoft は、Smart Password Lockout を使ってそのユーザー アカウントをロックします。  
https://docs.Microsoft.com/ja-jp/azure/active-directory/active-directory-secure-passwords#azure-ad-password-protections

## パスワードポシリーの強化
動的な禁止パスワードポリシー   
SSPRを利用することにより、攻撃対象となりやすいパスワードの利用を禁止 
https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-secure-passwords


## ADFSの保護
### ADFSブルートフォースアタック保護機能の有効化
ADFS利用の場合、Extranet Lockout Protectionを有効化することを強くお奨めします。
ADFS3.0(Windows Server 2012R2)以上が必要です。  
https://docs.microsoft.com/ja-jp/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-lockout-protection