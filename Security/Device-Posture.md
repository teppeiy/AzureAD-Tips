# 会社支給 vs BYOD ?

## 概要
会社支給デバイスのみに会社メールアクセス制限したい、BYODはブロックしたい、というニーズをよく聞きますが、デバイスの所有者によってアクセス制御するというのはセキュリティの観点からは、ほとんど意味はありません。信頼されたデバイスからのみ企業データへのアクセスを許可する、というのがセキュリティと利便性、そして運用管理コストのバランスをとる良い方法かもしれません。



## デバイスの所有者を区別することの意味は？
* セキュリティの観点  
多くの場合、意味はありません。例えば、個人が買うiPhoneと、会社の購買部門が買うiPhoneに物理的なハードウェアとしての違いがあるでしょうか？せいぜい会社所有物であることを表す資産管理シールが張ってあるぐらいです。

* 会社の就業規定・労務規定の観点  
これは会社によります。個人の資産を会社の企業活動へ利用してはならない、という会社の規定があるかもしれません。

## 重要なのはセキュリティポリシーに準拠していること
iPhoneから会社のデータ（Exchange Onlineのメール等）にアクセスさせる、というシナリオを考えます。
一般的に、情報漏えい等のセキュリティリスクを最小化するには、たとえば、「iPhoneをアンロックする際には6桁以上のPINが必要」、「10回間違ったらデバイス上のデータを削除する」等のセキュリティポリシーを適用します。技術的には、このようなポリシーはデバイスの所有者にかかわらず適用が可能です。
セキュリティポリシーに準拠しているかどうかをアクセス許可・拒否の判断材料にすることが本来の目的であることが多いですが、いつのまにか「会社支給デバイス以外は安全ではない」と、デバイス所有者の話にすりかわって解釈されているケースが多々見られます。

## セキュリティポリシーに準拠している = 信頼されたデバイス

Azure AD 条件付きアクセスでは、信頼されたデバイスであればアクセスを許可する、というアクセス制御が可能です。以下、Azure AD 条件付きアクセスが、信頼されたデバイスとみなすことができる管理手法です。詳しくは、[こちら](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-conditional-access-policy-connected-applications)を参照ください。

|管理状態|対象プラットフォーム|
|-|-|
|オンプレミスADドメイン参加していること（[ハイブリッドAzure AD 参加](https://docs.microsoft.com/ja-jp/azure/active-directory/device-management-hybrid-azuread-joined-devices-setup)）<br>=グループポリシー適用|Windows 7/8.1/10|
|Intuneのポリシーに準拠していること|iOS, Android, Windows 10, Windows Phone, MacOS|
|SCCMのポリシーに準拠していること<br>（[Intune Hybrid](https://docs.microsoft.com/ja-jp/sccm/mdm/deploy-use/setup-hybrid-mdm)）|MDM: iOS, Android, Windows 10, Windows Phone, MacOS <br> SCCM Agent: Windows 7/8.1/10
|Intune/SCCMのポリシーに準拠していること<br>（[Co-Management](https://docs.microsoft.com/ja-jp/sccm/core/clients/manage/co-management-overview)）|Windows 10|
|その他MDMのポリシーに準拠していること|Windows 10|

注意点： Azure AD参加していることだけでは、信頼のされたデバイス扱いにはならない
Windows 10 を Azure AD参加させたとしても、Azure AD 単体でセキュリティポリシーを適用するメカニズムを持っていないため、併せてIntune等の管理下におくことが必要です。


## 会社の規定により、BYOD利用を制限したい場合
* 会社の把握しているデバイス利用に制限したい  
[Intune の登録制限](https://docs.microsoft.com/ja-jp/intune/enrollment-options)を利用します。IMEI等のデバイス識別番号を管理者が予め登録し、そのデバイスのみをIntuneの管理下におくということが可能です。BYODを許可する場合においても、利用者が申請ベースでIMEIを事前に登録する（マニュアル作業）ことも可能です。
PCの場合には、オンプレミスADドメインに参加することを会社支給PCのみに制限することで実現可能です。

* BYOD利用許可により、従業員からの訴訟リスクが上がる  
従業員が深く考えずに個人所有デバイスをIntuneに登録し、業務メールを利用していたとします。子供のいたずら等で、PIN入力を規定回数以上に失敗し、個人のデータ（たとえば大切な子供の写真等）がワイプされた場合、企業は訴訟されるかもしれません。そういったリスクを避けるには、
[Azure AD Terms of Use](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-tou) が有効かもしれません。
[Azure AD Terms of Use](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-tou) を利用すると、従業員がサービス（Exchange Online等）へアクセスする際、使用条件の許諾を集めることが可能です。
