# オンプレ AD を無くしたい？ (WIP)
## 概要
クラウド利用があたりまえの時代になり、  
「オンプレ AD を無くして、全部クラウドで管理したい」   
「いつオンプレ AD を無くせるか？」   
などのご要望・ご質問をいただく機会が多くなっています。    
これに対し、実現可否やその時期について一般化して答えるのは難しいですが、オンプレ AD に依存しているリソースが多い企業では、最終的にオンプレ AD を無くすまで、5年、10年越しのプロジェクトになるかもしれません。  
また、オンプレ AD への依存度を下げていくことで、クラウドのメリットを享受できるため、早めに計画を立てて実行していくことをお奨めします。

## 推奨事項
クラウドのみでの管理に向けた準備を進める  
* 新規導入する Windows 10 は、Azure AD Join + Intune を利用した [モダンマネージメント](https://docs.microsoft.com/ja-jp/windows/client-management/manage-windows-10-in-your-organization-modern-management) をはじめる
* 新規導入するアプリケーションは、モダン認証（OpenID Connect、SAML、WS-Federation）対応を必須とし、Azure AD と認証連携する
* 既存のアプリケーションをクラウドへ移行する
* クラウド人事システム（HCM）の検討・導入する

では、どういう考え方で計画を進めていくのか、ということについてまとめます。

## オンプレ AD の役割の棚卸し
オンプレ AD への依存度を下げるにあたり、依存しているモノ（リソース）を Azure AD を中心としたクラウドサービス等に移動していきます。計画にあたり、以下のようなステップを踏みます。
1. オンプレ AD の役割を分解・整理
2. それらをクラウドで実現する場合の方法の整理

その場合、U/D/A モデルが役に立ちます。U/D/A とは、オンプレ AD に依存している以下のリソースの頭文字を取ったものです。
* (U)ser
* (D)evice
* (A)pplication

以下、それぞれについて、クラウド移行後のイメージや推奨・考慮事項について説明します。

## U（ユーザー）
WIP
<!-- 
### オンプレ AD の役割

* ユーザー情報（資格情報を含む）の保持  
* グループ情報の保持  

オンプレ AD は単純なデータベースで、ビジネスロジックは IDM 等の上位システムにある、というのが理想的な姿です。他の言い方をすると、仮にドメインコントローラーが全て壊れたとしても、ユーザーアカウントやグループが上位システムからリストアできるように設計されていることが重要です。  

### 考慮事項・推奨事項
前述のとおり、多くの企業において、オンプレ AD は、ユーザーアカウントライフサイクル (作成・変更・削除) の泉源ではない場合が殆どです。たとえば、社員が入社すると、先に人事システム等にユーザーのレコードが作られ、IDM 等のビジネスロジックを管理するシステムを通し、自動・手動でオンプレ AD にアカウントが作成される、というパターンです。単純化すると、以下のようなフローになるわけですが、このままではユーザープロビジョニングがオンプレ AD に依存し続けてしまいます。
 >人事システム → IDM → オンプレ AD → Azure AD   

最終的には、以下のようなフローに変更する必要があります。
 >人事システム → IDM → Azure AD

もし、人事システム (HCM) に Workday や、SuccessFactors を利用している場合、Azure AD の [インバウンドプロビジョニング機能](https://docs.microsoft.com/ja-jp/azure/active-directory/saas-apps/workday-inbound-tutorial) を利用することで、これを実現可能ですが移行過渡期においては、オンプレ AD も併用する必要があるため、何らかの形でオンプレ AD へのユーザープロビジョニングも必要になるはずです。例えば以下のようなフローが実現できればベストなのかもしれませんが、これは将来的にサポートするシナリオとして検討されている段階です。
 >人事システム → Azure AD (IDM) → オンプレ AD

-->
## D (デバイス)

### オンプレ AD の役割
デバイスの観点では、Windows を ドメイン参加 させ、グループポリシー（GPO）で管理する、というようなデバイス機構としての役割が大半を占めている場合がほとんどです。

### 考慮事項・推奨事項
新規導入する Windows 10 に対しては、[モダンマネージメント](https://docs.microsoft.com/ja-jp/windows/client-management/manage-windows-10-in-your-organization-modern-management) を検討します。簡単に言うと、Azure AD Join + Intune を利用し、クラウドのみで管理するという形態です。

#### 移行戦略
* 対象は Window 10 のみ
  * ダウンレベル OS （Windows 7 や 8.1）、サーバー OS は Azure AD Join および Intune での管理はサポートされていません。
  * サーバー OS については、アプリケーションのセクションで詳しく触れますが、アプリケーションの移行や Azure AD Domain Services を利用します。
* ドメイン参加 から Azure AD Join への移行パスは無い
  * IT 管理者による一括移行をすることができないため、エンドユーザーがドメイン参加 状態を解除し、その後、Azure AD Join をするという操作が必要になります。従って、現実的には新規導入する Windows 10 デバイスから Azure AD Join + Intune を利用することから始めることをお奨めします。
* Hybrid Azure AD Join から Azure AD Join への移行パスは無い
  * IT 管理者による一括移行をすることができないため、エンドユーザーがドメイン参加 状態を解除し、その後、Azure AD Join をするという操作が必要になります。
  * これは、Hybrid Azure AD Join をすべきではない、ということではありません。是非、既存の ドメイン参加 デバイスについては、Hybrid Azure AD Join を構成することをお奨めします。
  * マイクロソフトはこの移行パスの必要性や有効性を検証しています。
* Azure AD Join、Hybrid Azure AD Join 等の詳しい理解については、[Azure Active Directory のデバイス管理とは](
https://docs.microsoft.com/ja-jp/azure/active-directory/devices/overview) をご覧ください。

#### 運用
* Azure AD Join を会社支給 PC に限定するには？
  * AD 参加を会社支給 PC に限定する、という運用を行っている企業は少なくありません。この考え方をそのまま取り入れるのであれば、Intune で実現する [AutoPilot](https://docs.microsoft.com/ja-jp/windows/deployment/windows-autopilot/windows-autopilot) を利用します。
  * この際、会社支給 PC に縛る、という考え方を捨てることも検討してみるとよいかもしれません。セキュリティの観点では、会社の制御配下にあるデバイス、セキュリティポリシーが適用されているデバイス、であることが重要で、所有者は問題ではない場合がほとんどです。[会社支給 vs BYOD ?](https://github.com/teppeiy/AzureAD-Tips/blob/master/Security/Device-Posture.md) もご参考ください。
  * また、[Windows 10 を Intune へ登録するたくさんの方法 (英語)](https://microscott.azurewebsites.net/2018/08/31/managing-windows-10-with-intune-the-many-ways-to-enrol/) についても参考ください。
* デバイス調達、キッティングプロセスの見直し
  * 従来のイメージ型キッティングプロセスを見直し、 [AutoPilot](https://docs.microsoft.com/ja-jp/windows/deployment/windows-autopilot/windows-autopilot) を利用するにあたり、PC の調達経路についても考慮しなおす必要が出てきます。これに対応したハードウェアベンダーとの調整作業があります。

## A (アプリケーション)
### オンプレ AD の役割
アプリケーションの観点では、認証・認可のメカニズムを提供する、という役割があります。代表的な具体例として、以下が挙げられます。
* 統合 Windows 認証 を利用した、ファイルサーバー や IIS 等への認証機能の提供
* ファイルサーバー等へ、グループを利用した認可機能の提供
* LDAP を利用したユーザー・グループ属性情報等の提供

### 考慮事項・推奨事項
* 新規導入するアプリケーション
  * 調達要件として、モダン認証（OpenID Connect、SAML、WS-Federation）対応を必須とした上で、Azure AD と認証連携します。AD FS 等のオンプレ AD に依存する認証基盤を利用しないようにします。
* 既存のアプリケーション
  * アプリケーション整理し、ビジネスニーズに応じて取捨選択をします。自社ビジネスの戦略領域ではないビジネス要件については、可能な限りクラウドサービスの利用にシフトしていきます。例：ファイルサーバーから SharePoint Online 等。
  * 戦略領域である自社開発・運用のアプリケーションについては、改修ライフサイクルに応じてモダン認証に対応を進め、Azure AD と認証連携します。モダン認証対応については、[アプリケーションのモダン認証対応方法](MA-Apps.md) をご覧ください。
* 新規・既存アプリケーション共通
  * モダン認証対応については、[アプリケーションのモダン認証対応方法](MA-Apps.md) をご覧ください。
  * モダン認証に対応できないパッケージ製品、LDAP 等の古いプロトコルにたよざるをえないものについては、Domain Controller as a Service である、[Azure AD Domain Services](https://docs.microsoft.com/ja-jp/azure/active-directory-domain-services/active-directory-ds-overview) との連携を検討します。
