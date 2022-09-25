# allabout-ogawa-kubernetes
## 概要
学習のためにKubernetesでアプリを公開する。
業務で使われているKubernetesは、今まで使ったことがなかったため、実際に構築してみることで理解を深めたい。
今回は、公開することが目的なため、アプリはHello worldなど簡単なので構築する。極力、商用を意識したインフラ構造にする


## 要件を定義
### インフラ
- SSL証明書を設定
- ロードバランサを設定
- 複数zoneの冗長構成 (regionは1つでOK)
- 自動スケールなし (手動スケールの方法は調べておく)
- VPCは組まない (商用だとほぼ必須だが、目的からそれるため)

### アプリ
- 独自ドメインで公開
- アプリは既存のものを使う (Googleで公開されているレジストリを使う)
    - 「gcr.io/google-samples/hello-app:1.0」を利用


## 準備
### ローカル開発環境
- OS
    - macOS
- ツール
    - docker
    - docker-compose
    - kubectl
    - gcloud

### GCP環境
- GCPの Kubernetes Engine APIを有効にしておく

    ```
    1. 課金を有効
    2. Artifact Registry and Google Kubernetes Engine API を有効
    ```

- ローカルで、gcloudログインしておく

    ```
    gcloud auth login
    ```

- 固定IP作成

    ```
    gcloud compute addresses create <固定IP名> --global
    ```

- 固定IP確認

    ```
    gcloud compute addresses describe <固定IP名> --global
    ```

- DNSの設定

    ```
    独自ドメインは持っているものを使う
    上記のIPをAレコードで設定しておく
    ```

## 手順
### クラスターの設定
- kubernetesクラスタ作成

    ```
    cluster_name=allabout-ogawa-kubernetes
    gcloud container clusters create $cluster_name --machine-type=e2-micro --num-nodes=1 --region=us-west1
    ```

- クラスタの認証情報を取得 (このコマンドで、以下で指定したクラスターにデプロイされるようになる)

    ```
    cluster_name=allabout-ogawa-kubernetes
    gcloud container clusters get-credentials $cluster_name
    ```

### クラスターにデプロイ (マニフェスト利用するver)
- namespace.yamlをapplay
- cert.yamlをapply (プロビジョニングには時間がかかる)
- deployment.yamlをapply
- service.yamlをapply
- ingress.yamlをapply

```
kubectl apply -f namespace.yaml
kubectl apply -f cert.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```

### 稼働確認

```
https://kubernetes.ogawa.allabout.oootaiji.com/
```


## GKEのmanifestファイルについて
- cert.yaml: 証明書の設定
- deployment.yaml: デプロイの仕様
- service.yaml: Serviceの仕様
- ingress.yaml: Ingressの仕様
- namespace.yaml: namespaceを作成するためのyaml


## 参考文献
- [kubectlインストール](https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/#homebrewを使用してmacosへインストールする)
- [gcloudインストール](https://cloud.google.com/sdk/docs/quickstart)
- [Kubernetes Engine APIの有効化](https://cloud.google.com/kubernetes-engine/docs/quickstart)
- [GKEデプロイ](https://qiita.com/8yoshiyoshi/items/99a16843e081979ff627)
- [GKEにhello worldをデプロイ](https://medium.com/@gorlemkun/gkeにhelloとだけ言わせたい-google-kubernetes-engineに無をデプロイする-7c8feea1921c)
- [SSL化してデプロイ](https://cloud.google.com/kubernetes-engine/docs/how-to/managed-certs)