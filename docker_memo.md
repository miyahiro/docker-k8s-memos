# Docker講座メモ

## 参考情報

[Udemy講座：元Microsoftエンジニアが教える、ドッカーの理論と実践コース。Docker（docker-compose含む）を1から学びを図解で解説。また現場目線のベストプラクティスとサンプルWebアプリ構築のコーディングチャレンジも含む。](https://uniadexjp.udemy.com/course/linux-docker-compose-dockerfile-kanzennyumon/learn/lecture/20662064
)

[本家Dockerドキュメント（ja）](https://docs.docker.jp/index.html)

## DockerのCommand Lines

```sh
# Dockerに関する情報を表示する
docker info

# Docker Hubにログインする
# あんまり使わない
docker login

# Docker Hubからイメージ`image_name`をダウンロードする
docker pull image_name

# Dockerイメージ`image_name`の例らーを確認する
docker hisotry image_name

# Dockerイメージ一覧を確認する
docker images

# Dockerイメージ`image_name`を削除
docker rmi image_name

# Dockerコンテナをイメージ`image_name`から`container_name`という名前のコンテナを作成し、コンテナのを開始する
# -p 8080:80 : ホスト状の8080とコンテナ内の80ポートをマッピング
# -v $(pwd):/app/conf : ホスト上の$(pwd)を/app/confにマウントする
# --env key=value : コンテナ内に環境変数`key`を値として`value`で設定
# -d : コンテナを起動させるが、ttyはでタッチする
# --rm : コンテナ停止時に自動的にコンテナを削除する
# --name container_name : 起動するコンテナの名前を`container_name`とする
docker run \
  -p 8080:80 \
  -v $(pwd):/app/conf
  --env key=value \
  -d \
  --rm \
  --name container_name \
  image_name
  
# コンテナ一覧を表示する
# -a : 停止しているコンテナも表示する
# -q : 詳細情報は表示しない
docker ps -a
docker ps -a -q

# コンテナ`container_name`に`sh`プロセスを作成し、そのプロセスとホスト上のシェルのttyを接続する
# -itは--interactive --ttyの略で、対話式(interactive)でttyに接続するを意味する.
docker exec -it container_name sh

# コンテナ`container_name`を停止する
dokcer stop container_name

# コンテナ`container_name`を開始する
dokcer start container_name

# Dockerコンテナ`container_name`のメタ情報を表示する
docker inspect container_name

# Dockerコンテナ`container_name`のログを表示する
# -f : ログを表示し続ける(followする)、tail -f と同じ感覚
docker logs container_name

docker logs container_name -f

# 起動しているコンテナ`container_name`から`image_name`を起動する
docker commit container_name image_name

# コンテナを削除する
docker rm container_name

# Dockerfileからコンテキストとして`.`(カレントディレクトリ)としてDockerイメージを`image_name`として作成する
# --tag image_name : 作成後のDockerイメージの名前を`image_name`とする
# . : Dockerイメージを作成する際のコンテキストをカレントディレクトとする
# -f MyDockerfile: Dockerイメージを作成する際の設定ファイル(なにも指定しない場合は`Dockerfile`という名前になる)として`MyDockerfile`を使う.
docker build --tab image_name .

docker build --tab image_name -f MyDockerfile .

# すでにあるDockerイメージ`image_name`に`new_image_name`という名前(タグ)を付与する
docker --tag image_name new_image_name

# Docker HubにDockerイメージ`image_name`をpushする
# イメージ名に制限あり
# Docker Hubにログインしていないと失敗する
docker push image_name

# Dockerで使用していないものをいろいろと削除してくれる
# pruneは刈り込むという意味
docker system prune

# Dockerのネットワーク情報に関するコマンド
# Dockerのnetwork modeとしては以下がある
# bridge, host, null, overlay
# bridge: ブリッジネットワーク(仮想的なBridgeインターフェース(NIC)を使用)。ifconfigで確認できる？
# host: ホストのネットワーク(NIC)を共有
# null: ネットワークには接続しない
# overlay: よくわからないが、複数ホスト上のコンテナが同じネットワークに所属させるために必要となる機能らしい.
docker network ls
docker network create --driver bridge bridge_network_name
docker network rm bridge_network_name
docker run --network=none --rm -d --name none -p 80:80 nginx

# その他
docker rm -vf
docker ps -a -q
dicker images -a -q
docker rmi -f $(docker images -a -q)
docker rm -vf $(docker ps -a -q)

# Docker contaienr
docker-compose --file docker_compose_file.yaml up -d
docker-compose --file docker_compose_file.yaml down -d
docker-compose -f docker_compose_file.yaml logs --follow

# 3 docker containers for nginx service in docker-compose yaml file.
# Other service is 1 
docker-compose -f docker_compose_file.yaml up --scale nginx=3 -d
```

## Docker file

### Dockerfile解説-1

```Dockerfile
# FROMでベースとなるイメージを指定
FROM nginx:latest 

# docker containerがコマンドを開始するディレクトリ(ワーキングディレクトリ)を指定
WORKDIR /usr/share/nginx/html

# COPYでホストからファイルをイメージにコピー
COPY index.html index.html 

# RUNでシェルコマンドを実行
RUN apt update && apt install -y curl

# 他にもWORKDIR,ENV,CMDなど変数があります
```

### DockerfileのTips

```sh
# パーミッションの変更
# RUN chown -R node:node /app && \
#     chmod -R 744 /app && \
#     chown -R node:node /node_modules && \
#     chmod -R 777 /node_modules
# パーミッション変更するとイメージサイズが増えるので、COPY時に以下のようにしてchownする.
COPY --chown=node:node . .

# Dockerのコンテナの実行ユーザの変更
USER node

# PORTという環境変数を設定
# 起動するプログラムが`PORT`という環境変数を読み取って
# オープンするポートを変えることが前提
ENV PORT 8080

# 8080を使うことを宣言
EXPOSE 8080

# &&で数珠つなぎにするとイメージサイズを抑えられる
RUN npm ci \
  && npm cache clean --force \
  && mv /app/node_modules /node_modules

```

## Docker Compose files

### Docker Compose解説-1

```yaml
# Docker Composeファイルのバージョンを指定
version: "3.7"

# 構成するサービス一覧を定義
services:
  # voteというサービスの定義
  vote:
    # Dockerイメージをビルドするときのコンテキストを`./vote`とする
    # `./vote`配下に`Dockerfile`という名前のDockerイメージを作成するための設定ファイルを置いておく必要がある.
    build: ./vote
    # コンテナを起動したときに実行するコマンドライン
    command: python app.py
    # マウントするボリュームの設定
    volumes:
      # ホスト上の`./vote`とコンテナ内の`/app`をマウント
      - ./vote:/app
    # ポートマップ定義
    ports:
      # ホストの`5000`とコンテナの`80`をマッピングする
      - "5000:80"
    # 所属するネットワークを指定する
    # ここで設定する内容はトップレベルの`networks`キー配下の設定値を指定する
    networks:
      - front-tier
      - back-tier

  # resultというサービスの定義
  result:
    build: ./result
    command: nodemon server.js
    volumes:
      - ./result:/app
    ports:
      - "5001:80"
      - "5858:5858"
    networks:
      - front-tier
      - back-tier

  # workerというサービスの定義
  worker:
    build:
      # ビルドコンテキストを明示的に`context`キーで指定する
      context: ./worker
    # 依存している他のサービス
    depends_on:
      - "redis"
      - "db"
    networks:
      - back-tier

  # radisというサービスの定義
  redis:
    # 使用するイメージ
    image: redis:alpine
    # コンテナ名を明示的に指定
    container_name: redis
    # ホストの6379とコンテナの6379をマッピングするときにはこのようにも書ける
    ports: ["6379"]
    networks:
      - back-tier

  # dbというサービスの定義
  db:
    image: postgres:9.4
    container_name: db
    # 環境変数を指定
    # 環境変数は環境変数名をキー、環境変数の値をバリューとして指定する
    environment:
      POSTGRES_USER: "postgres"
      POSTGRES_PASSWORD: "postgres"
    volumes:
      # トップレベルの`volumes`キー配下で設定しているボリューム`db-data`をコンテナ内の`var/lib/postgresql/data`にマウントする
      - "db-data:/var/lib/postgresql/data"
    networks:
      - back-tier

# ボリュームの設定
volumes:
  # `db-data`という名前のDockerボリュームを作成する
  # このボリュームの実体は...
  db-data:

# docker network create --driver bridge front-tier
# ネットワークの設定
networks:
  # `front-tier`という名前のネットワークを作成
  # ここでの名前を`services`配下の`networks`で指定する
  front-tier:
    # ネットワークの種類を指定
    # bridgeの場合、わざわざdriverキーで指定しなくてもよい(デフォルトがbridgeなので)
    driver: bridge # default value, can be ommitted
    # ネットワーク名(docker network ls で表示されるネットワーク名)を指定（多分）
    name: front-tier  # specify name here otherwise folder name ("example-voting-app") will be prepended to network name "example-voting-app_front-tier"
  back-tier:
    driver: bridge # default value, can be ommitted
    name: back-tier # specify name here otherwise folder name ("example-voting-app") will be prepended to network name "example-voting-app_back-tier"
```

## fragments

```yaml
# docker-compose version.
version: 3.7
```

```yaml
# サービス一覧
services:
  # `radis`というサービスを設定.
  radis:
```

```yaml
services:
  xxx:
    # build設定
    build:
      # buildコンテキストを`./worker`に指定
      context: ./worker
```

```yaml
services:
  xxx:
    # Dockerイメージ指定
    image: "redis:alpine"
```

```yaml
services:
  xxx:
    # docker-composeコマンドで`docker-compose --profile seed up`
    # と`--profile`を指定して起動した場合のみに起動されるサービス
    profile: ["seed"]
```

```yaml
services:
  xxx:
    # リスタートするか.
    # 設定用の処理を動かすだけのDockerについては`restart`を`"no"`に設定し、
    # さらに`profile: ["seed"]`などのようにしておくと、
    # `--profile seed`を指定して`docker-compose`を実行したときに、
    # 設定処理用のDockerコンテナが起動し、終了、再起動はしないというようなことができる.
    restart: "no"
```

```yaml
services:
  xxx:
    # コマンドを上書き
    # このサービスを起動したときに、`python app.py`を実行せよと指定している.
    command: python app.py
```

```yaml
services:
  xxx:
    # エントリポイントを上書き
    # `commnad`の指定と`entrypoint`の指定については違いがあるようだが、わかっていない.
    # 実際に使う時に調査すること.
    entrypoint: nodemon server.js
```

```yaml
services:
  xxx:
    # 依存しているサービスを指定
    depends_on:
      # `radis`サービスに依存
      radis:
        # service_healthyという状態であればOK
        condition: service_healthy
```

```yaml
services:
  xxx:
    # ヘルスチェック設定
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"],
      interval: 15s
      timeout: 5s
      retries: 3
      start_period: 10s
```

```yaml
services:
  xxx:
    # 接続するボリュームマウント設定
    volumes:
    # ホスト上のディレクトリとのマッピング
    - "./vote:/app"
    # トップレベルの`volume`で作成したボリュームとのマッピング
    - "db-data:/var/lib/postgresql/data"
```

```yaml
services:
  xxx:
    # ポートマッピング
    ports:
    # ホストの"8080"とコンテナ内の"80"をマッピング
    - "8080:80"
```

```yaml
service:
  xxx:
    # 環境変数設定
    environments:
      key1: value1
      key2: value2
```

```yaml
services:
  xxx:
    # 接続するネットワーク設定
    networks:
    # トップレベルの`networks`で作成したネットワーク`back-tier`と接続.
    # `docker network create --driver bridge back-tier`
    # と同じ.
    - back-tier
    # トップレベルの`networks`で作成したネットワーク`front-tier`と接続.
    - front-tier
```

```yaml
# ボリューム設定
volumes:
  # `db-data`という名前のボリュームを作成
  db-data:
```

```yaml
# ネットワーク設定
networks:
  # `front-tier`という名前のBridgeネットワークを作成
  front-tier:
  # `back-tier`という名前のBridgeネットワークを作成
  # 明示的にネットワークの種別`bridge`と名前を指定.
  back-tier:
    driver: bridge
    name: back-tier
```

### サンプル-Support Rreplica

```yaml
version: 3.7
services:
  nginx:
    image: nginx:latest
    # Not Specify container name to support replica.
#    container_name: docker_compose_nginx
    environment:
    - env_key=env_value
    ports:
#    - 80:80
    # Rangeで指定
    - 80-85:80
    volumes:
    - ${PWD}:/usr/share/nginx/html:ro
  ubuntu:
    image: ubuntu
#    container_name: docker_compose_ubuntu
    command:
    - sleep
    - "1500"
    ports:
#    - 8080:8080
    - 8080-8085:8080
  busybox:
    image: busybox
#    container_name: docker_compose_busybox
    command:
    - "sleep"
    - "1500"
```
