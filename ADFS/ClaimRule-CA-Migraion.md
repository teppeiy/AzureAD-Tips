# アクセス制御のモダナイゼーション
AD FSのクレームルールから条件付きアクセスへの移行

## 概要
アクセスポリシーを1つの場所で集中管理することを
特に、AD FSのクレームルールからAzure ADの条件付きアクセスへ移行する際のポリシー設計や、考慮ポイントについて触れます。

## ステップ
1. 現状把握
2. アクセス制御要件の整理
3. 条件付きアクセスポリシーの設計
4. 検証
5. 本番環境への適用
6. モニタリング

## 1. 現状把握
現状のアクセスポリシーを把握し、アクセス制御ポリシーの場所は以下のように多岐に渡っている可能性があります。
* ADFS等のフェデレーションサービス
    * ADFSの場合には、[ADFS Config Dump](ADFS/ADFS-Config-Dump.md) を利用して ADFS 構成情報の出力が可能
* Azure AD
    * 既存の条件付きアクセス設定の整理
* アプリ
    * 例: Salesforceなどの SaaS アプリケーション側でのアクセス制御など

## 2. アクセス制御要件の整理
Azure AD でのアクセス制御を行うにあたり、再度自組織で必要なアクセス制御要件を整理します。例えば以下を要素とし、登場人物とその利用シーンのパターンを定義します。
* 利用するユーザー (社員 (役員、一般社員、契約社員)、非社員など)
* 利用する場所 (社内/外、国など)
* 利用するデバイス (社給、BYOD/CYOD など)
* 利用するアプリケーション (Exchange Online, Skype for Business, Salesforce など)
* 利用するクライアントアプリケーション (Office アプリ, Web ブラウザ, SaaS アプリ専用ネイティブアプリ など)
多くの検討要素がありますが、これらを組み合わせたパターンをなるべく少なく整理することが要件整理のコツです。ここで整理された利用パターンごとにアクセス時にどのような制御 (許可、拒否、条件付きで許可 (多要素認証など)) をしたいかを整理します。


## 3. 条件付きアクセスポリシーの設計
[条件付きアクセス](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-conditional-access-azure-portal)の理解  
[Azure Active Directory の条件付きアクセスのベスト プラクティス](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-conditional-access-best-practices)  


#### ベースポリシーの決定
ベースポリシー = 全体の８割以上の利用ケースに該当する条件  
推奨：場所を問わず、(MDM等の)ポリシーに準拠したデバイスからのアクセスを許可


#### 除外ケースの抽出

一般的な除外ケース  
| 対象 | シナリオ | 対処 (実装) |  
|---|---|---|
| 全体管理者 | 管理者自身をロックアウトさせないよう | 全体管理者を全ポリシー除外 (Global Administrator Role を除外する方法はPrivate Preview中) |  
| Azure AD Connect 同期アカウント| MFA/デバイスポリシー等の制御の影響を受け、同期が失敗するリスクを軽減 | 同期アカウントを全ポリシーから除外 |
| ゲストアクセス | SPO/Teams等、社外ユーザーにIP制限等でブロックしないよう | ゲストをまとめた動的グループを全ポリシーから除外 (動的グループ無しでGuestを除外する方法はPrivate Preview中) |
| MS以外のモバイルアプリ |  |  |

全体管理者等の特権アカウントは、一般ユーザーのポリシーからは除外しますが、別な方法で必ず保護してください。 詳しくは、[Azure ADのセキュリティ強化](Security/Secure-AzureAD.md) を参照。

[動的グループでゲストアカウントを括る方法](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-b2b-dynamic-groups)

### 先進認証 (Modern Auth) の有効化

[Office クライアントで Office 365 先進認証を使用する](https://support.office.com/ja-jp/article/Office-クライアントで-Office-365-先進認証を使用する-776c0036-66fd-41cb-8928-5495c0f9168a)
[Exchange Online で先進認証を有効または無効にする](https://support.office.com/ja-jp/article/exchange-online-%E3%81%A7%E5%85%88%E9%80%B2%E8%AA%8D%E8%A8%BC%E3%82%92%E6%9C%89%E5%8A%B9%E3%81%BE%E3%81%9F%E3%81%AF%E7%84%A1%E5%8A%B9%E3%81%AB%E3%81%99%E3%82%8B-58018196-f918-49cd-8238-56f57f38d662)

[Skype for Business Online で先進認証を有効または無効にする](https://social.technet.microsoft.com/wiki/contents/articles/34339.skype-for-business-online-enable-your-tenant-for-modern-authentication.aspx)

[Office 製品の Office 365 モダン認証フローと認証キャッシュについて](
https://blogs.technet.microsoft.com/sharepoint_support/2016/08/01/modern-authentication-flow-and-cache-of-office-to-office-365/)

有効化に伴う考慮ポイント

* 先進認証＝ブラウザ認証フロー になるため、既存のクレームルールの制御が変わる可能性がある
* 社外からのアクセスが許可されるケースがある
    * 一度社内でリソース(ExO等)へアクセス、その後、社外へPCを持ち出しても当分の間利用できてしまう
    http://azuread.net/2017/07/28/%E5%85%88%E9%80%B2%E8%AA%8D%E8%A8%BC%E5%88%A9%E7%94%A8%E6%99%82%E3%81%AEadfs%E3%81%AB%E3%82%88%E3%82%8B%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E5%88%B6%E5%BE%A1%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6/


#### 場所ベースの制御のために
社内・社外は、信頼できるIPアドレスの範囲を名前付き場所として定義  
https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-named-locations

ADFSのInsideCorporateNetworkクレームを利用する方法もあるが、近い将来ADFSを撤廃することを考えると名前付き場所の利用を推奨  
https://docs.microsoft.com/ja-jp/azure/multi-factor-authentication/multi-factor-authentication-get-started-adfs-cloud


[What-If分析](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-conditional-access-whatif)
