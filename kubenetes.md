# 用語

1. pod

    Kubenetesの管理の最小単位
    複数のコンテナから構成される
    ネットワーク(NIC/IPアドレス)がPod内のコンテナで共有される
    ボリューム(ファイルシステム)がPod内のコンテナで共有される
    
# 知識

1. 同一クラスタ内のPod間の通信は可能.

# コマンドライン

```sh
# helloworldという名前のpodをイメージgcr.io/google-samples/hello-app:1.0から作成
# podを作成する場合は --restart Never を設定する
kubectl run --image gcr.io/google-samples/hello-app:1.0 --restart Never helloworld

# pod一覧を取得
kubectl get pods

# pod(helloworldという名前内のコンテナのログを表示
kubectl logs helloworld

# podのメタデータを表示
kubectl describe pod helloworld

# pod内のコンテナにshell接続
kubectrl exec -it helloworld sh

# podを削除
kubectl delete pod helloworld

# Pod内のコンテナの環境変数を定義
kubectl run --env TEST_ENV=hello_world \
  -- image gcr.io/google-samples/hello-app:1.0 \
  --restart Nevwer helloworld

# Pod内のコンテナのポートをマッピング
kubectl run --port 8080:8080 \
  -- image gcr.io/google-samples/hello-app:1.0 \
  --restart Nevwer helloworld
  
# curlという名前のpodを curlimages/curl:7...68.0 というイメージを
# もとに作成し、それにsh接続する
# --rm とすることで、shが終了してコンテナが終了したら、コンテナを削除される
kubectl run --restart Never \
  --image curlimages/curl:7.68.0 -it --rm curl sh
```
