# XSS_Nginx
nginxでのXSS対応の調査。

レスポンスヘッダに下記を付与する。

- X-XSS-Protection
- Content-Security-Policy

## 設定

```/etc/nginx/conf.d/default.conf```に記載する。

``` conf : /etc/nginx/conf.d/default.conf
    # X-XSS-Protectionを強制的に付与する
    add_header X-XSS-Protection "1; mode=block";
```

### curl結果

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

## 起動

```
docker-compose up -d
```

## URL

[http://localhost:8080/index.php](http://localhost:8080/index.php)

## 参考

- [Nginxセキュリティ設定:Qiita](https://qiita.com/hideji2/items/1421f9bff2a97a5e5794)