# Azure AD アプリケーションプロキシのFAQ（ウェブアプリ観点）

## 概要

アプリケーションプロキシを利用する際、よく聞かれる質問と回答を記載します。

### Q1. 公開したページが文字化けする

[リンク変換](https://docs.microsoft.com/ja-jp/azure/active-directory/application-proxy-link-translation
)を利用する場合、アプリケーションが UTF-8 でエンコードされていることを前提としています。 そうなっていない場合には、http 応答ヘッダーでエンコードの種類を指定してください。

``` HTML
Content-Type:text/html;charset=utf-8
```

### Q2. 公開したウェブアプリのセッション情報引き継ぎがうまくいかない

Cookieを利用してセッションを保持するウェブアプリを公開した際、Cookieが引き継がれないためセッション情報（例：ログイン状態等）がページ間で引き継がれない場合があります。
その場合、[CSRF](https://www.ipa.go.jp/security/awareness/vendor/programmingv2/contents/301.html)への対策等で、ウェブアプリからのCookie発行時（Set-Cookie）に特定ドメインがスコープとして指定されていないか確認してください。ブラウザの開発者モード（F12）やFiddlerが役に立ちます。

``` HTML
Set-Cookie: SSO_COOKIE=xxx; path=/; domain=.contoso.local
```

上記の例は、社内URLが指定されたケースですが、もし以下のような構成の場合、発行されたCookieのスコープ外である社外URLへアクセスした際、そのCookieは利用されません（ブラウザはリクエストの際にCookieを送信しない）。

* 社外URL： app1-contoso.msappproxy.net
* 社内URL： conotoso.local

対処方法としては、2点あります。

1. 社内URL＝社外URLにする （[カスタムドメイン](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-application-proxy-custom-domains)の利用）
2. 1が難しい場合、ウェブアプリを改修し、[Cookieのスコープ](https://developer.mozilla.org/ja/docs/Web/HTTP/Cookies)を変更する

### Q3. JavaScriptを利用しているページが想定どおり動かない

Q2のCookieのケースのように、社外URLと社内URLが異なる場合にドメイン名をまたがるリクエストが発生してしていて、[CSRF](https://www.ipa.go.jp/security/awareness/vendor/programmingv2/contents/301.html)への対策に影響受けていないか確認します。ブラウザの開発者モード（F12）やFiddlerが役に立ちます。

対処方法としては、2点あります（Cookieのケースとほぼ同じ）。

1. 社内URL＝社外URLにする （[カスタムドメイン](https://docs.microsoft.com/ja-jp/azure/active-directory/active-directory-application-proxy-custom-domains)の利用）
2. 1が難しい場合、ウェブアプリを改修し、[CORS](https://developer.mozilla.org/ja/docs/Web/HTTP/HTTP_access_control)を考慮・実装する
