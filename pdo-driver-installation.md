# bibouroku-docker > pdo-driver-installation

文書作成日:西暦2023年6月29日 最終更新日:西暦2023年7月19日

Docker公式のPHP-fpmのイメージには、SQLite用のPDOドライバーしか用意されていない。このためMySQLやPostgreSQLを利用する場合、別途導入作業が必要となる。PostgreSQLを使用する場合の手順について書き留める。

1. [導入手順](#導入手順)
2. [注意点](#注意点)

## 導入手順
PDOドライバはモジュールや拡張機能と呼ばれるものに相当するので`docker-php-ext-install`コマンドを用いる。

以下の文章をコンテナにするDockerfileに記述する。

「Dockerfile内の記述」
```
RUN apt-get update \
    && apt-get install -y libpq-dev \
    && docker-php-ext-install pdo_pgsql
```

`apt-get`はDebian系OSで使われるパッケージ管理に関するコマンド。`pdo_pgsql`が入手する対象のドライバであり、ここではPostgreSQL用のドライバを指定している。つまり、この記述はMySQLなど使うデータベースに応じて変える。

## 注意点
ドライバー名に「`_`」を用いていることに注意。コマンドに使われている「`-`」と間違えやすい。

`libpq-dev`の記述は必須。この記述がない場合は以下のエラーが発生する。

「libpqの不足によるエラー」
```
configure: error: Cannot find libpq-fe.h. Please specify correct PostgreSQL installation path
```

## 参考文献
1. https://hub.docker.com/_/php (php - Official Image | Docker Hub)
