# iPadだけでプログラミング環境を整えた話[VPS+Coder]

Created: 2021年7月2日 14:26
Updated: 2023年2月14日 8:46
Published_Time: 2023年2月14日
Slug: programming-with-coder
Tags: iPad, 開発環境
Published: Yes
URL: https://blog.shaba.dev/posts/programming-with-coder

## はじめに

GithubCodespacesがまだ無かった時代。

なんとかiPadを開発デバイスにしたく、試行錯誤した末にたどり着いた当時の開発環境を公開します。

なお、現在はこちらの記事のとおりCodespacesを使っています。

とても便利でオススメ。

[iPadでプログラミング開発環境構築の努力と2023年個人的ベストプラクティス[Blink+VSCode+Codespaces]](https://blog.shaba.dev/posts/coding-with-ipad)

## Coderとは

[Coder](https://coder.com/)とは、Codespacesと同じく、ブラウザ上でVSCodeライクな操作感で開発をすることができるサービスです。

クラウドサービスですが、セルフホストで使うことも出来て、その場合は無料です。

[Coder - Remote development on your infrastructure](https://coder.com/)

サービス公開用のVPSを既に借りていたので、そこでホストすればいいやということで、今回はセルフホストすることにしました。

![F7635320-BD2A-4608-B6B3-4AAEB417F5E4.jpeg](F7635320-BD2A-4608-B6B3-4AAEB417F5E4.jpeg)

実際にこんな感じで開発をしていました。

（その後、色々辛いところが出てきてvimでの開発に移行しましたが。。。）

## 運用するまでにやったこと

### インストール

詳しい手順は忘れてしまったのですが、当時と今（2023年2月）とはまた話が違いそうなので、[github](https://github.com/coder/coder)を参照ください。

### ドメインの取得

当方webアプリを開発したかったので、開発中の画面を確認できる必要がありました。

なので、Coder用とpreview用の2つのサブドメインを作成していました。

ローカル環境を立ち上げる際のポート番号を固定して、ポートフォワーディングすることで実現していました。

Codespacesも似たような仕組みですね。

### セキュリティ周りの設定

流石に開発環境と開発画面が誰でも見られる状態だとヤバいので、最低限の認証でbasic認証を入れました。

が、webkitの昔からある現象？で、 websocket使用時にbasic認証がうまく動かないらしく。

[80362 - WebSocket: Client does not support 401 Unauthorized response.](https://bugs.webkit.org/show_bug.cgi?id=80362)

Coderはwebsocketを使用していて、iPadのブラウザはすべてwebkitなのでbasic認証が効かず、しばらく頭を抱えました。

仕方ないので、code-server側は当時もう一つ気になっていたAuth0でOAuth化することにしました。

oauth2を導入する手順はこちらのサイト様がとてもわかり易かったです。

[oauth2_proxy と Auth0 を用いた Nginx のお手軽 OAuth 化 · Yutaka 🍊 Kato](https://mikan.github.io/2018/05/23/enable-oauth-to-your-nginx-by-oauth2-proxy-and-auth0/)

メンバを絞るなら、

1. rules -> create rule-> whitelistでemailを絞る
2. Users&roles ->Users->create Usersでユーザ作成
3. Connections->Database->Disable SignUpsを有効

という手順でできそう。

### SSL化

SSL化もします。おなじみLet’s Encryptですが、設定頻度が高くないので毎回やり方を忘れがちです。

使い方は下記サイト様が分りやすかったです。

[【無料SSL入門】「Let's Encrypt」とは？設定で挫折しない！使い方解説 - カゴヤのサーバー研究室](https://www.kagoya.jp/howto/it-glossary/security/lets-encrypt/)

### nginx設定

上記を踏まえたnginxの設定がこちらです。

ポート番号1111がOAuth認証用、2222がCoder用、3333がpreview用として使い分けています。

```
map $http_upgrade $connection_upgrade {
default upgrade;
'' close;
}
server {
server_name code.domain.com;
location /oauth2/ {
    proxy_pass http://localhost:1111;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Scheme $scheme;
    proxy_set_header X-Auth-Request_Redirect $request_uri;
}

location = /oauth2/auth {
    proxy_pass http://localhost:1111;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Scheme $scheme;
    proxy_set_header Content-Length "";
    proxy_pass_request_body off;
}

location / {
    #oauth2
    auth_request /oauth2/auth;
    error_page 401 = /oauth2/sign_in;
    auth_request_set $user $upstream_http_x_auth_request_user;
    auth_request_set $email $upstream_http_x_auth_request_email;
    proxy_set_header X-User $user;
    proxy_set_header X-Email $email;
    auth_request_set $auth_cookie $upstream_http_set_cookie;
    add_header Set-Cookie $auth_cookie;

    proxy_pass http://127.0.0.1:2222;
        #websocket
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-NginxX-Proxy true;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;
        proxy_redirect off;

    }
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/code.from-garage.work/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/code.from-garage.work/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

server {
    server_name preview.domain.com;
    location /webhook {
        proxy_pass http://127.0.0.1:3333;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    location / {
        #Basic auth
        auth_basic "Restricted"; # 認証時に表示されるメッセージ
        auth_basic_user_file /etc/nginx/.htpasswd;

        proxy_pass http://127.0.0.1:3333;
        proxy_set_header Host $http_host;
       proxy_redirect off;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/preview.from-garage.work/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/preview.from-garage.work/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

server {
    if ($host = code.domain.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    listen 80;
    server_name code.domain.com;
    return 404; # managed by Certbot
}

server {
    if ($host = preview.domain.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    listen 80;
    server_name preview.domain.com;
  return 404; # managed by Certbot
}
```

### 1プロジェクトの構成

だいたいこんな感じの構成になっていました。

紹介程度に。

```python
├── code_config_root
│ └── config.yaml
├── docker
│ ├── code-server
│ │ ├── app
│ │ ├── config
│ │ ├── config_backup
│ │ ├── Dockerfile
│ │ └── ssh
│ ├── mysql
│ │ ├── docker-entry-point-init.db
│ │ └── my.cnf
│ └── Dockerfile
├── docker-compose.yml
├── nginx.conf
├── src
│ ├── app1
│ └── app2
└── volume
 └── mysql
 ├── auto.cnf
 └── …
```

## 試して没になったもの

### browser preview

Coderで入れたが単体では動かず、google-chromeとchrome driverがホスト内に必要だとわかったので入れてみましたが、

dockerを立ち上げてからbrowser-previewが動くようになるまで数分かかったのと、スクロールできない致命的な現象を確認したので断念しました。

今入れたらどうかはわかりません。もしかしたらVPSのスペックの問題かも知れません。

- google-chrome
    
    ```bash
    RUN wget -q -O – https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add && \
    echo ‘deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main’ | tee /etc/apt/sources.list.d/google-chrome.list && \
    apt-get update && \
    apt-get install -y google-chrome-stable
    ```
    
- ChromeDriver
    
    ```bash
    ADD https://chromedriver.storage.googleapis.com/77.0.3865.10/chromedriver_linux64.zip /opt/chrome/
    RUN cd /opt/chrome/ && \
    unzip chromedriver_linux64.zip
    ```
    

## 終わりに

現在はより便利なCodespacesに移ってしまっていますが、実際にこの構成でアプリを作って（少額ながらも）お金が発生したので、とても思い入れが有ります。

セルフホストするためにいろんなサービスや設定を網羅しないといけなかったので、大変でしたがとても良い勉強になったと思っています！