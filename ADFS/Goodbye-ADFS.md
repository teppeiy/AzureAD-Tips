# ADFS廃止のすすめ

## 概要
これまで、Office 365等へのシングルサインオンやアクセス制御をADFSで実施するというケースが主流でしたが、最近ではAzure ADへのシフトが加速化されています。
1. オンプレミス環境の管理から解放
ADFSを利用する場合、HA/DRが前提だと最低4台（ADFS/WAP x 2）また、テスト・QA環境も含めると更に多くのサーバーの管理が必要です。そして、各種証明書の管理、DMZを含む外部からのアクセスセキュリティの管理、ロードバランサーの管理、セキュリティ更新プログラムの管理等、比較的大きなコストと労力が必要ですが、これから開放されます。
2. Azure ADの条件付きアクセス等の最新機能が利用可能
非常に複雑な構文を要するADFSのクレームルールを管理することは手間がかかりますし、変更管理やテストにかかる労力や変更適用までの時間は小さくありません。条件付きアクセスを利用することで、GUIを使った分かりやすい形でのアクセスポリシーを管理でき、
3. xxx

## ADFS撤廃までのステップ概要


### パスワードハッシュ同期の有効化
パスワードハッシュ同期は、ADFSを無くすためのステップの一つですが、2つのメリットをもたらします。
1. 漏洩した資格情報検知レポートによるセキュリティ向上
マイクロソフトは、複数のソースから漏洩した資格情報一覧を継続的に取得しています。そのリストとAzure ADアカウントと機械的に突き合わせることにより、漏洩した資格情報を検知しレポートを提供しています。
2. ディザスタリカバリへの対策
フェデレーション環境において、ADFS/WAPが利用できなくなった際には、Office 365等Azure ADに認証を依存しているサービスが利用できなくなります。実際にADFSがPetyaに感染した企業において、パスワードハッシュ同期をしていた企業はADFSからAzure ADへの認証に切り替えることによって、数時間のダウンタイムでサービスへの認証アクセスが復旧できました。一方、パスワードハッシュ同期をしていなかったため、ADFSを構築しなおす等の作業で、数日間のダウンタイムを強いられた企業もあります。

#### パスワードハッシュをAzure ADに同期することの安全性について
オンプレミスActive Directoryには、MD4でハッシュ化されたパスワードが保存されています。パスワードハッシュ同期を有効化すると、Azure AD Connectを介し、そのハッシュ化されたパスワードを更に（Salt +）ハッシュ化した上でAzure ADへ同期します。
https://docs.microsoft.com/ja-jp/azure/active-directory/connect/active-directory-aadconnectsync-implement-password-synchronization#how-password-synchronization-works

#### ステップ:
1. Azure AD Connectの構成ウィザードを利用し、パスワードハッシュ同期を有効化する
https://docs.microsoft.com/ja-jp/azure/active-directory/connect/active-directory-aadconnectsync-implement-password-synchronization#enable-password-synchronization
2. 漏洩した資格情報検知レポートを確認する
https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-reporting-security-user-at-risk

#### Azure AD のパスワード保護
* Smart Lockout
ユーザーのパスワードがサイバー犯罪者によってハッキングされようとしている可能性があることを Azure AD が検出すると、Microsoft は、Smart Password Lockout を使ってそのユーザー アカウントをロックします。
https://docs.Microsoft.com/ja-jp/azure/active-directory/active-directory-secure-passwords#azure-ad-password-protections

#### 更にセキュリティレベルを上げる
* Identify Protection によるリスクベースでの条件付きアクセスの有効化
https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-identityprotection

* Ignite 2017 - Shut the door to cybercrime with Azure Active Directory risk-based identity protection
https://myignite.microsoft.com/sessions/53404?source=sessions


### ADFSに依存しているサービスを移行
#### 移行要件の確認
移行難易度・可否を判断する上での考慮ポイント
* [ADFS Config Dump](ADFS-Config-Dump.md)による移行判断レポート
* ライセンス
* ADFSでしか実現することができない要件の確認
[ADFS Config Dump](ADFS-Config-Dump.md)による移行判断レポート
Azure AD開発部門で開発した[ADFS Config Dump](ADFS-Config-Dump.md)により取得したADFSの設定情報から、Relying Party（証明書利用者信頼）の移行判定レポートを提供します。

#### よくある移行障壁について
1. EncryptClaims
トークンを暗号化している状態にあります。現状Azure ADではサポートされていない機能です。本当にトークンの暗号化が必須要件であるか再度検討をお奨めいたします。
また、Azure ADでも将来的にトークン暗号化をサポートする計画はあります。
2. IssuanceAuthorizationRule（クレーム発行許可ルール）
いわゆるクレームルールで、アクセス制御している状態にあります。Azure ADの条件付きアクセスを利用することで、更に柔軟で管理のしやすいアクセス制御をすることが可能です。移行における大部分の労力はクレームルール⇒条件付きアクセスの移行に費やされることになります。

#### ライセンス
ADFSでクレームルールを利用し、場所ベースのアクセス制御をしている利用ケースは少なくありません。これと同様あるいは更に高度な機能な有するAzure AD条件付きアクセスを利用することになりますが、Azure AD Premium P1ライセンスが必要になります。これまで、あまりITシステムの利用が限られているユーザー（例：工場勤務の従業員等）がライセンスを保持していないケースは、追加購入の検討が必要になります。

#### ADFSでしか実現することができない要件の確認
Azure ADの継続的な機能向上に伴い、ADFSでしか実現できないシナリオは急速に減りつつありますが、まだ、いくつか気を付けるべきポイントが存在します。
https://blogs.msdn.microsoft.com/samueld/2017/06/13/choosing-the-right-sign-in-option-to-connect-to-azure-ad-office-365/


### クレームルール から条件付きアクセスへ移行
#### 条件付きアクセス
https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-conditional-access-azure-portal
#### メリット
より柔軟なアクセス制御
将来的にはリスクベースのアクセスの適用
#### 考慮ポイント
* ライセンス – 要Azure AD Premium P1
*技術的要件（ADFSでしかできないこと）
* レガシー認証のブロック → CY2017中に実現
* レガシー認証の条件付きアクセス → CY2018中に実現
https://blogs.msdn.microsoft.com/samueld/2017/06/13/choosing-the-right-sign-in-option-to-connect-to-azure-ad-office-365/


#### 移行戦略
* 移行対象グループがADFSではなくCAを利用している状態を確立し、移行対象グループのメンバーを徐々に増やしていく
* クレームルールがバイパスされるGroup Aがあるため、その配下に「CA Users（仮称）」グループを作成し、そのグループをCA対象グループとする
##### この方式の問題点
オンプレADでCA Usersグループに対象アカウントを登録し、Azure AD Connectでグループメンバーシップ情報がAzure ADへ同期されるまでの間（最大30分程度、同期周期に依存）、ADFSのクレームルールはバイパス、条件付きアクセスの対象にならない（＝アクセス許可）という状態が存在する
リスク軽減方法 => CA Usersグループにメンバーを追加した直後に同期をする等