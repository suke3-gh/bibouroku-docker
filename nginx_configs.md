# bibouroku-docker > nginx-configs

文書作成日:西暦2023年6月29日 最終更新日:西暦2023年7月19日

Docker上でnginxを動作させる過程で苦労した点と、それに対する解決策を記述し記録する。

1. [default.conf関連](#defaultconf関連)
    1. [location ~ \.php-内の記述](#location--php-内の記述)
2. [volumesに記述しているディレクトリ](#volumesに記述しているディレクトリ)
3. [参考文献](#参考文献)

## default.conf関連
- Nginxのコンテナにおけるこのファイルの位置は`/etc/nginx/conf.d/default.conf`。
- `http {}`の部分は使用できない。変更したい設定は`server {}`に記述していく。
- `root`の指定は必須のようだ。これが正しくないとファイルが想定していたパスで見つからなくなる。現状`/usr/share/nginx/html/`を指定することで想定通りの動作をしている。
記述位置はserverブロック直下。locationブロック内などに記述すると動作に問題が見られた。
- `index`を指定しなかった場合、indexファイルの拡張子が`.html`なら内容が表示されるが`.php`では403エラーとなる。こちらも指定すべき。

### location ~ \.php$ {}内の記述
`location ~ \.php$ {}`内の`fastcgi_index`は不要でも動作に問題はなさそうだ。一方で、fastCGIでのPHPを利用するには次の記述は必須。  

「location ~ \.php$ {}内の記述」
```
fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
fastcgi_param  QUERY_STRING    $query_string;
fastcgi_pass   php-fpm:9000;
include        fastcgi_params;
```

- `fastcgi_pass`のコロン記号左辺は用いるサービス名を記述する。初期状態だと`localhost`やそれを表すIPアドレスになっている。
- `fastcgi_param  SCRIPT_FILENAME`の初期値は`/script$fastcgi_script_name;`になってるが、dockerでは上記のものに書き換えたほうが良いようだ。

## volumesに記述しているディレクトリ
以下 `docker-compose.yml` が存在するディレクトリを相対パスの起点として記述する。

NginxはWebなどに公開するファイルが置かれているディレクトリを`/usr/share/nginx/html/`としているようだ。つまりこれをコンテナ側のマウント先として設定する。そしてホスト側のマウント先を記述し、そこに各種ファイルを置くことでWebサーバ上にそれらのファイルが存在するに等しい状態となる。コロン記号左辺にホスト側、右辺にコンテナ側のパスを記述する。

「volumesの記述例」
```
volumes: './nginx/web/:/usr/share/nginx/html/'
```

## 参考文献
1. https://hub.docker.com/_/nginx (nginx - Docker Hub)
2. http://nginx.org/en/docs/beginners_guide.html (Beginner's Guide - Nginx)
