# 特定アプリへのアクセス時に必ずID/Passを入力させたい

## 概要
Azure ADと認証連携をしたアプリへのアクセス時に、あえて毎回認証させたい場合、アプリが発行するAzure ADへの認証要求にパラメータを引き渡すことで実現可能です。

### SAML-Pのアプリ
SAML2.0プロトコルを利用する場合、SAML Request内にForceAuthn="true"を指定します。

SAML Requestの例
```XML
<samlp:AuthnRequest
xmlns="urn:oasis:names:tc:SAML:2.0:metadata"
ID="id6c1c178c166d486687be4aaf5e482730"
Version="2.0" IssueInstant="2013-03-18T03:28:54.1839884Z" ForceAuthn="true"
xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
<Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">https://www.contoso.com</Issuer>
</samlp:AuthnRequest>
```

https://docs.microsoft.com/ja-jp/azure/active-directory/develop/active-directory-single-sign-on-protocol-reference#authnrequest


### OpenID Connectのアプリ
OpenID Connectプロトコルを利用する場合、サインイン要求にprompt=loginを指定します。

サインイン要求の例
```
GET https://login.microsoftonline.com/{tenant}/oauth2/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=id_token
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&response_mode=form_post
&scope=openid
&state=12345
&prompt=login
&nonce=7362CAEA-9CA5-4B43-9BA3-34D7C303EBA7
```

https://docs.microsoft.com/ja-jp/azure/active-directory/develop/active-directory-protocols-openid-connect-code#send-the-sign-in-request

尚、 OpenID Connect 利用の際に追加の多要素認証を求めたいというケースでは上記のサインイン要求に加えて amr_values=mfa を与えることで Azure MFA による多要素認証を求めることができます。ただし、Azure AD 開発部門では将来的に acr_values パラメーターを受け付けるようにする機能改修を予定しており、amr_values パラメーターからの移行を推奨する可能性があります。
