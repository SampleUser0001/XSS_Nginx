# XSS_Nginx
nginxでのXSS対応の調査。

レスポンスヘッダに下記を付与する。

- X-XSS-Protection
- Content-Security-Policy

## 設定

### X-XSS-Protection

```
HTTP の X-XSS-Protection レスポンスヘッダーは Internet Explorer, Chrome, Safari の機能で、反射型クロスサイトスクリプティング (XSS) 攻撃を検出したときに、ページの読み込みを停止するためのものです。
```
引用元：[https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/X-XSS-Protection](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/X-XSS-Protection)


```/etc/nginx/conf.d/default.conf```に記載する。

``` conf : /etc/nginx/conf.d/default.conf
    # X-XSS-Protectionを強制的に付与する
    add_header X-XSS-Protection "1; mode=block";
```

#### curl結果

設定前

``` txt
HTTP/1.1 200 OK
Server: nginx/1.19.9
Date: Wed, 14 Apr 2021 04:22:43 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
X-Powered-By: PHP/7.2.34
```

設定後

``` txt
HTTP/1.1 200 OK
Server: nginx/1.19.9
Date: Wed, 14 Apr 2021 04:23:32 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
X-Powered-By: PHP/7.2.34
X-XSS-Protection: 1; mode=block
```

### Content-Security-Policy

```
コンテンツセキュリティポリシー (CSP) は、クロスサイトスクリプティング (XSS) やデータインジェクション攻撃などのような、特定の種類の攻撃を検知し、影響を軽減するために追加できるセキュリティレイヤーです。これらの攻撃はデータの窃取からサイトの改ざん、マルウェアの拡散に至るまで、様々な目的に用いられます。
```
引用元：[https://developer.mozilla.org/ja/docs/Web/HTTP/CSP](https://developer.mozilla.org/ja/docs/Web/HTTP/CSP)

```/etc/nginx/conf.d/default.conf```に記載する。

``` conf : /etc/nginx/conf.d/default.conf
    # Content-Security-Policyの付与
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://ssl.google-analytics.com; img-src 'self' https://ssl.google-analytics.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com 'self' https://themes.googleusercontent.com; frame-src 'none' object-src 'none'";
```
設定値は[Nginxセキュリティ設定:Qiita](https://qiita.com/hideji2/items/1421f9bff2a97a5e5794)からそのままコピペした。  
実際のシステム構成に合わせて値変更すること。

#### curl結果

設定前

``` txt
HTTP/1.1 200 OK
Server: nginx/1.19.9
Date: Wed, 14 Apr 2021 04:23:32 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
X-Powered-By: PHP/7.2.34
X-XSS-Protection: 1; mode=block
```

設定後

``` txt
HTTP/1.1 200 OK
Server: nginx/1.19.9
Date: Wed, 14 Apr 2021 04:31:07 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
X-Powered-By: PHP/7.2.34
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://ssl.google-analytics.com; img-src 'self' https://ssl.google-analytics.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com 'self' https://themes.googleusercontent.com; frame-src 'none' object-src 'none'
```

## 起動

```
docker-compose up -d
```

## URL

[http://localhost:8080/index.php](http://localhost:8080/index.php)

## 参考

- [Nginxセキュリティ設定:Qiita](https://qiita.com/hideji2/items/1421f9bff2a97a5e5794)
- [X-XSS-Protection:MDN Web Docs](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/X-XSS-Protection)
- [コンテンツセキュリティポリシー (CSP):MDN Web Docs](https://developer.mozilla.org/ja/docs/Web/HTTP/CSP)
- [Content-Security-Policy:MDN Web Docs](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Content-Security-Policy)
