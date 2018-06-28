# レガシー認証をブロックする

## 概要
Azure AD 条件付きアクセスで、レガシー認証をブロックすることが可能になりました。


## レガシー認証とその問題
レガシー認証は基本認証とも呼ばれます。たとえば、POP3、IMAP4、SMTP等のプロトコルを利用する際に使われます。
サービス仲介型の古い認証フローを利用することから、特に、**多要素認証が利用できない**、というのが大きな問題です。パスワードだけでは防ぎきれない昨今の攻撃に対処することができません。
基本認証と先進認証の両方に対応したプロトコル（例：MAPI、EWS等）もあります。


## これまでのブロック方法
1. ADFSのクレームルールで対応  
この方法が利用されているケースを非常に多く見られます。クラウド認証（PHS/PTA）では利用できない、また、ADFSを撤廃したいニーズに答えられないというのが問題でした。
2. ユーザーへAzure MFAを強制、かつ、アプリパスワードの利用を拒否
この方法だと、その他の先進認証を利用するサービスへのアクセスの際、毎回多要素認証が要求されてしまう、という点が問題でした。

## 2018年6月の Azure AD 機能拡張
1. サインインログで、レガシー認証の利用状況等が確認できるようになりました  
サインインイベント毎に、対象となった条件付きアクセスポリシー、アクセス元の詳細（クライアント/プロトコル/レガシー認証かどうか等）を確認することが可能になりました。詳しくは[こちら](#)（準備中）をご覧ください。
2. 条件付きアクセスで、レガシー認証をブロックすることが可能に  
詳しくは[こちら](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-conditional-access-conditions#legacy-authentication)をご覧ください。


## ステップ
1. サインインログで確認する  
[Azure Portal](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/SignIns) へ管理者権限（セキュリティ閲覧者以上）でログインし、「サインイン」ブレードへ移動  
自社テナントでレガシー認証の利用状況を確認し、ブロックすることの影響を調査します。

2. 条件付きアクセスを設定する
[条件付きアクセスブレード](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ConditionalAccessBlade/Policies)へ移動し、ポリシーを設定します。
必ず、[こちら](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-conditional-access-conditions#legacy-authentication)に記載のある既知の問題やFAQに目を通していただくことを強くお奨めします。ActiveSync等には利用可能な条件に制限があります。

## 注意事項
上記公式ドキュメントの既知の問題やFAQに加え、気をつけるべきポイントを挙げます。  

レガシー認証を利用するプロトコル（POP3/IMAP等）でOutlookからExchange Onlineへアクセスする際、そのプロトコルではOS情報が必ずしも付帯されて送信されません。こういったことから、Azure ADはOS情報を知る術がないということになり、「デバイスプラットフォーム」条件に合致せず、意図するポリシーが適用されないことがあります。

対象方法としては、デバイスプラットフォーム条件にて「すべてのプラットフォーム (サポート対象外を含む)」を利用し、OS情報が認識されない場合にもポリシーが適用される（条件が合致する）ようにします。レガシー認証関連のポリシーのみではなく、一般的な条件付きアクセスポリシーを利用する際の一般的なベストプラクティスです。

「デバイスプラットフォーム」条件を100％信頼したポリシーデザインは、セキュリティ観点で好ましくありません。たとえば、利用者がアクセス元User-Agentを偽り、OS情報をご認識させてIT管理者が意図するポリシーをバイパスされてしまう可能性はゼロではありません。条件付きアクセスのOS情報の認識方法は、Azure ADが認識できるあらゆる情報（非公開）を使うことで精度を高めていますが、前述したようなリスクがゼロではないということをご認識ください。


<!--
## レガシー認証を悪用した攻撃とその対策
Azure AD 条件付きアクセスの他にも、防御策として適したものがいくつかあります。

クラウド認証のケース
|認証フロー|攻撃|防御策|備考|
|---|---|---|---|
|1. メールクライアントがExOへU/Pを送る|攻撃者はブルートフォースやパスワードスプレー攻撃で使えるU/Pを検証する|[ExOで基本認証を無効にする](https://support.office.com/ja-jp/article/disable-basic-authentication-in-exchange-online-bba2059a-7242-41d0-bb3f-baaf7ec1abd7)|備考|
|2. ExOがAzure ADへU/Pを送る|攻撃者はブルートフォースやパスワードスプレー攻撃で使えるU/Pを検証する|スマートロックアウト (Azure ADの基本機能)|備考|
|3. Azure ADがExOへ認証結果を返す|攻撃者は正しいU/Pでデータ（アドレス帳等）を盗む|New Azure AD CA|備考|
|||[ExO Client Access Rule](https://technet.microsoft.com/ja-jp/library/mt842508(v=exchg.150).aspx)|備考|


フェデレーション（ADFS等）認証のケース
|認証フロー|攻撃|防御策|備考|
|---|---|---|---|
|1. メールクライアントがExOへU/Pを送る|攻撃者はブルートフォースやパスワードスプレー攻撃で使えるU/Pを検証する|[ExOで基本認証を無効にする](https://support.office.com/ja-jp/article/disable-basic-authentication-in-exchange-online-bba2059a-7242-41d0-bb3f-baaf7ec1abd7)|備考|
|2. ExOがIdPへU/Pを送る|攻撃者はブルートフォースやパスワードスプレー攻撃で使えるU/Pを検証する|スマートロックアウト (Azure ADの基本機能)|備考|
|3. Azure ADがExOへ認証結果を返す|攻撃者は正しいU/Pでデータ（アドレス帳等）を盗む|New Azure AD CA|備考|
|||[ExO Client Access Rule](https://technet.microsoft.com/ja-jp/library/mt842508(v=exchg.150).aspx)|備考|

U/P: Username/Password  
ExO: Exchange Online
-->