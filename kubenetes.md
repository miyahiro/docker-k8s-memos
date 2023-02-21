# 用語

1. Pod

    Kubenetesの管理の最小単位
    複数のコンテナから構成される
    ネットワーク(NIC/IPアドレス)がPod内のコンテナで共有される
    ボリューム(ファイルシステム)がPod内のコンテナで共有される
    
2. Volume

    データを格納するストレージリソース.
    種類としては以下がある

    - emptyDir
    
        このタイプのボリュームは永続化されない.
        Podが停止すると勝手に消えてしまう.
    
    - hostPath
    
        Podが動作するホストのディレクトリとマッピングするタイプのボリューム.
        Podが停止しても起動しないが、Podが起動するホストが変わるとマウントするボリュームが異なる.

    - configMap
    - secret
    - gcePersistentDisk
    - awsElasticBlockStore
    - csi
    - downwardAPI
    - nfs

    gcePersistentDisk, awsElasticBlockStoreはクラウドサービスなどを使用してVolumeを作成してマウントする方法.

    minikubeでは、emptyDirやhostPathが使用できる.
    
3. ReplicaSet

    ReplicaSetは指定された数のPodを複製し、実行してくれる.
    Podのひな形(Pod Templates)を定義し、Label Selectorという方法で管理対象を選択してReplicaSetに制御してもらう.
    要点としては以下の3つ.

    - replicas: Podの複製数
    - selector: Label Selector(どのポッドを動作させるかを選択するためのもの)
    - template: Podのひな形(Pod Template)を指定.
    
    ReplicaSetをapplyすると、ReplicaSetリソースだけではなく、ReplicaSetの定義をもとに
    作成されたPodも作成される.
    
    ReplicaSet単体で使用することは少なく、Deploymentリソースを作成したときに付随してReplicaSetリソースが作成されることが多い.

    ```sh
    kubectl apply -f replicaset_01.yaml
    kubectl get all
    ```

    を実行すると、以下のような出力が得られる.

    ```txt
    NAME              READY   STATUS    RESTARTS   AGE
    pod/nginx-4qkst   1/1     Running   0          13s
    pod/nginx-mhv6s   1/1     Running   0          13s
    pod/nginx-w9dbv   1/1     Running   0          13s

    NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
    service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   20d

    NAME                    DESIRED   CURRENT   READY   AGE
    replicaset.apps/nginx   3         3         3       13s
    ```
    
    また、replicaset_01.yamlの`replicas`の値を変更して `kubectl apply -f replicaset_01.yaml` を実行すると、その設定合わせてPod数を増やしてくれる.
    
    また、`kubectl scale --replicas=1 -f replicaset_01.yaml`としてもPod数を操作できる.

4. Deployment

    Deploymentは以下の機能を提供するリソース.

    - PodのRolling Update
    - Rolloutの履歴を保持
    - Rollback
    
    DeploymentはPodを直接管理しているのではなく、ReplicaSetを管理することで、間接的にPodを管理している.
    Dploymentを変更した際は、新しいReplicasetを作成してPodを置き換えていく.
    
    Dploymentで重要となるフィールドは以下

    - minReadySeconds
    - paused
    - progressDeadlineseconds
    - revisionHistoryLimit
    - strategy
    
    その他

    - replicas
    - selector
    - template

    があるが、これらのフィールドはReplicaSetと同様のものとなる.
    
    

5. Service

    Podをクラスター内外に公開する静的IPを持ったL4ロードバランサー.  
    クラスター内外からPodへの安定的なアクセスを提供できる仮想IPアドレスをServiceに割り当てる.
    
    Serviceには以下の3つのタイプがある

    1. ClusterIP
    2. NodePort
    3. LoadBalancer
    
6. Ingress

    L7(http?)のロードバランサー.
    
### Podのライフサイクル

    1. Pending
    
        Kubenetesによって承認されたが、１つ以上のコンテナが作成されていない状態.ImageのPullやinitContainerの処理中の場合この状態になる.

    2. Runing
    
        コンテナが作成され、少なくとも1つのコンテナが実行されているか監視・再起動中の状態
        
    3. Succeeded
    
        Pod内のすべてのコンテナが正常終了し、再起動されていない状態
    
    4. Failed

        Pod内のすべてのコンテナが終了し、少なくとも1つ以上のコンテナの終了ステータスが0意外だったか、Kubenetesによって強制終了させられた場合

    5. Unknown
    
        Podの状態を取得できなかった際の状態
        
    コンテナのライフライクル発生時に実行する処理を設定できる.

    - postStart

        コンテナが作成された後、即座に実行される.このHandlerが失敗した場合終了させ、restartPoolicyに従い再起動させる

    - preStop

        コンテナが終了される前に実行される.コンテナのルートプロセスに対してSIGTERMが送出される前にこのHandlerが実行される.
        
    preStopが定義されているとそれを実行するが、この処理が先に終了してしまうとコンテナ側が処理中だとしてもKubenetesはコンテナに対してSIGTERMを送出する.
    preStopとは別にコンテナの終了処理を行っている最中にSIGTERMが送られると望まれない結果となることがあるので注意.
    これを回避する溜めには、preStop側でSleepをさせるなどの大気処理が必要.
    preStopの処理の最大期間はterminationGracePeriodSecondsやkubectlで指定されたGracePeriodの値に依存するらしい?
    
    postStartやpreStopに指定できるHandlerの種類は以下.

    - exec

        コンテナ内でコマンドを実行

    - httpGet

        HTTPのGETリクエストを発行(どこから？)
    
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
# --rm: shが終了してコンテナが終了したら、コンテナを削除される
# --restart Never: こうするとPodが作成される
kubectl run --restart Never \
  --image curlimages/curl:7.68.0 -it --rm curl sh
```

----

```sh
# minikubeで降雨地区下ローカル環境のVMにsshで接続
minikube ssh
```

```sh
# create k8s resource from file.
kubectl create -f pod-nginx.yaml

# get pod info
kubectl get pod

# get k8s all resources info
kubectl get all

# delete pod
kubectl delete pod/nginx

# yamlファイルを適用する
kubectl apply -f nginx.yaml

# Podを削除して再作成する
kubectl replace --force -f date-tail.yaml

# Podを削除する
kkubectl delete -f data-tail.yaml

# pod nginx で `ps`コマンドを実行する
kubectl exec nginx -- ps

# Pod date-tail の date コンテナで ls /var/log/date-tail というコマンドを実行
kubectl exec -it date-tail -c date -- ls /var/log/date-tail

# Pod date-tail のコンテナ tail のログをfloow(-f)しながら、10個取得(--tail=10)
kubectl logs date-tail -c tail -f --tail=10

# どのように実行されているかをわかりやすく表示する
# initContainerを指定した場合の確認コマンドとして登場
kubectl get po -w

# replicaset_01.yamlのreplicasの値を1に設定してapplyする.
kubectl scale --replicas=1 -f replicaset_01.yaml

```