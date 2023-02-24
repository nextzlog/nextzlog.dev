---
title: How to run Contest Administration System
subtitle: コンテスト運営システムATS-4の起動方法
---

## 解説動画

<div class='ratio ratio-16x9'>
<iframe src='https://www.youtube.com/embed/Yb6QY7BI4kA?vq=hd1080' title='YouTube video player' allowfullscreen></iframe>
</div>

## 起動方法

まず、[Docker](https://docs.docker.jp/desktop/install.html)を用意します。
次に、以下のスクリプトをコピーして、ターミナルに貼り付けて、実行します。

```sh
cat << EOS > docker-compose.yaml
version: '3'
services:
  ATS4:
    image: ghcr.io/nextzlog/ats4:master
    ports:
    - 9000:9000
    volumes:
    - ./ats/data:/ats/data
    - ./ats/logs:/ats/logs
    - ./ats.conf:/ats/conf/ats.conf
    - ./rules.rb:/ats/conf/rules.rb
    command: /ats/bin/ats4
  www:
    image: nginx:latest
    ports:
    - 80:80
    volumes:
    - ./proxy.conf:/etc/nginx/conf.d/default.conf
EOS

echo -n 'enter mail hostname: '
read host

echo -n 'enter mail username: '
read user

echo -n 'enter mail password: '
read pass

cat << EOS > ats.conf
play.mailer.host=$host
play.mailer.port=465
play.mailer.ssl=true
play.mailer.user="$user"
play.mailer.password="$pass"
play.mailer.mock=false
ats4.rules=/rules.rb
EOS

cat << EOS > rules.rb
require 'rules/sample/plain'
RULE
EOS

echo -n 'enter server domain: '
read name

cat << EOS > proxy.conf
server {
  server_name $name;
  location / {
    proxy_pass http://ATS4:9000;
    location /admin/ {
      allow 127.0.0.1;
      deny all;
    }
    location ~ /admin {
      allow 127.0.0.1;
      deny all;
    }
  }
}
EOS

docker compose up -d
```

これで、[ローカルホスト](http://localhost)でATS-4が起動します。
個別のコンテストの規約に合わせた詳細な設定方法は、無線部開発班にご相談を。
