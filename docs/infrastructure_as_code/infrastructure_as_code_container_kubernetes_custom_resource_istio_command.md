---
title: 【IT技術の知見】コマンド＠Istio
description: コマンド＠Istioの知見を記録しています。
---

# コマンド＠Istio

## はじめに

本サイトにつきまして、以下をご認識のほど宜しくお願いいたします。

ℹ️ 参考：https://hiroki-it.github.io/tech-notebook-mkdocs/about.html

<br>

## 01. セットアップ

### インストール

#### ▼ brewリポジトリから

```bash
$ brew install istioctl
```

#### ▼ GitHubリポジトリから

ℹ️ 参考：https://istio.io/latest/docs/setup/getting-started/#download

（１）インストール先のディレクトリに移動する。

```bash
$ cd /Users/hiroki.hasegawa/projects
```

（２）GitHubリポジトリからインストールする。

```bash
$ curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.12.1 - sh
```

（３）istioctlへのパスを環境変数に登録する。

```bash
$ cd istio-1.12.1
$ export PATH=$PWD/bin:$PATH
```

<br>

### プロファイル

#### ▼ プロファイルとは

Istioの機能のセットを提供する。

ℹ️ 参考：https://istio.io/latest/docs/setup/additional-setup/config-profiles/

#### ▼ プロファイルの種類

ℹ️ 参考：

- https://atmarkit.itmedia.co.jp/ait/articles/2111/05/news005.html
- https://betterprogramming.pub/getting-started-with-istio-on-kubernetes-e582800121ea

| ユースケース         | default  | demo     | empty                         | external | minimal              | openshift | preview | remote |
| :------------------- | :------- | :------- | :---------------------------- | -------- | :------------------- | --------- | ------- | ------ |
| 概要                 | 本番環境 | 開発環境 | Istioリソースを全てカスタマイズしたい | -        | 最小限の機能が欲しい | ？        | -       | ？     |
| istio-egressgateway  | -        | ○        | -                             | -        | -                    | ？        | -       | ？     |
| istio-ingressgateway | ○        | ○        | -                             | -        | -                    | ？        | ○       | ？     |
| istiod               | ○        | ○        | -                             | -        | ○                    | ？        | ○       | ？     |

<br>

## 02. istioctlコマンド

### analyze

#### ▼ analyzeとは

Istioが正しく機能しているか否かを検証する。

ℹ️ 参考：https://istio.io/latest/docs/reference/commands/istioctl/#istioctl-analyze



成功した場合を以下に示す。

```bash
$ istioctl analyze

✔ No validation issues found when analyzing namespace: default.
```

失敗した場合を以下に示す。

```bash
$ istioctl analyze

Info [IST0118] (Service default/foo-service) Port name  (port: 80, targetPort: 80) doesn't follow the naming convention of Istio port.
```

#### ▼ -n

Namespaceを指定しつつ、```analyze```コマンドを実行する。

```bash
$ istioctl analyze -n <Namespace名>
```

<br>

### destroy

#### ▼ istioctlコマンドを使用して

Istioリソースを全てdestroyする。

```bash
$ istioctl x uninstall --purge
```

<br>

### install

#### ▼ installとは

プロファイルをインストールし、また設定値を変更する。

ℹ️ 参考：https://istio.io/latest/docs/setup/install/istioctl/

#### ▼ -f

IstioOperatorのマニフェストファイルを使用して、プロファイルをインストールする。

ℹ️ 参考：https://istio.io/latest/docs/setup/install/istioctl/#install-istio-using-the-default-profile

```bash
$ istioctl install -y -f <IstioOperatorのマニフェストファイルへのパス>
```

#### ▼ --set

インストールするもの、または変更する項目を指定する。

ℹ️ 参考：https://istio.io/latest/docs/setup/additional-setup/config-profiles/

| オプション例                               | 説明                                                         | 補足                                                         |
| ------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ```meshConfig.accessLogFile=/dev/stdout``` | アクセスログの出力先を標準出力に変更する。                   |                                                              |
| ```profile=default```                      | 指定したプロファイルをインストールする。                     |                                                              |
| ```revision=n-n-n```                       | 既存のIstioのコントロールプレーンを稼働させつつ、指定したバージョンのコントロールプレーンをカナリアリリースする。バージョンは、ケバブケースで設定する必要がある。 | ℹ️ 参考：https://istio.io/latest/docs/setup/upgrade/canary/#control-plane |

<br>

### kube-inject

#### ▼ kube-injectとは

istio-proxyコンテナをサイドカーコンテナとして作成する。代わりに、```enabled```値が割り当てられた```istio-injection```タグをNamespaceに付与しても良い。

ℹ️ 参考：https://istio.io/latest/docs/reference/commands/istioctl/#istioctl-kube-inject

```bash
$ istioctl kube-inject
```

<br>

### manifest diff

#### ▼ diffとは

ymlファイルの差分を取得する。

ℹ️ 参考：https://istio.io/latest/docs/reference/commands/istioctl/#istioctl-manifest-diff

```bash
$ istioctl manifest diff <変更前マニフェストファイルへのパス> <変更後マニフェストへのパス>
```

<br>

### operator

#### ▼ init

IstioOperatorを```istio-system```にapplyする。

```bash
$ istioctl operator init

Installing operator controller in namespace: istio-operator using image: docker.io/istio/operator:1.12.1
Operator controller will watch namespaces: istio-system
✔ Istio operator installed
✔ Installation complete
```

<br>

### profile

#### ▼ profileとは

Istioのプロファイルを操作する。

ℹ️ 参考：https://istio.io/latest/docs/reference/commands/istioctl/#istioctl-profile

#### ▼ list

利用できるプロファイルを取得する。

```bash
$ istioctl profile list

Istio configuration profiles:
    default
    demo
    empty
    external
    minimal
    openshift
    preview
    remote
```

<br>

### proxy-config

#### ▼ proxy-config

Istio上で管理されるEnvoyの構成情報を取得する。

ℹ️ 参考：

- https://istio.io/latest/docs/reference/commands/istioctl/#istioctl-proxy-config
- https://sreake.com/blog/istio/

```bash
$ istioctl proxy-config <設定項目> <Pod名> -n <Namespace名>
```

Envoyのエンドポイント情報を取得する。

```bash
$ istioctl proxy-config endpoints <IngressGateway名> -n istio-system

ENDPOINT                         STATUS      OUTLIER CHECK     CLUSTER
127.0.0.1:15000                  HEALTHY     OK                prometheus_stats
127.0.0.1:15020                  HEALTHY     OK                agent

# 〜 中略 〜

172.17.0.11:9090                 HEALTHY     OK                outbound|80||kubernetes-dashboard.kubernetes-dashboard.svc.cluster.local
172.17.0.13:80                   HEALTHY     OK                outbound|80||foo-service.microservices-with-kubernetes.svc.cluster.local

# 〜 中略 〜

192.168.64.14:8443               HEALTHY     OK                outbound|443||kubernetes.default.svc.cluster.local
unix://./etc/istio/proxy/SDS     HEALTHY     OK                sds-grpc
unix://./etc/istio/proxy/XDS     HEALTHY     OK                xds-grpc
```

Envoyのリスナー情報を取得する。

```bash
$ istioctl proxy-config listeners <IngressGateway名> -n istio-system

ADDRESS PORT  MATCH DESTINATION
0.0.0.0 8080  ALL   Route: http.8080
0.0.0.0 15021 ALL   Inline Route: /healthz/ready*
0.0.0.0 15090 ALL   Inline Route: /stats/prometheus*
```

のルーティング情報を取得する。

```bash
$ istioctl proxy-config routes <IngressGateway名> -n istio-system

NAME          DOMAINS     MATCH                  VIRTUAL SERVICE
http.8080     *           /*                     foo-virtual-service.istio-system
              *           /stats/prometheus*     
              *           /healthz/ready*  
```

<br>

### proxy-status

#### ▼ proxy-statusとは

IngressGateway、EgressGateway、istio-proxyコンテナのステータスを取得する。

ℹ️ 参考：https://istio.io/latest/docs/reference/commands/istioctl/#istioctl-proxy-status

```bash
$ istioctl proxy-status  

NAME                                      CDS        LDS        EDS        RDS          ISTIOD           VERSION
istio-egressgateway-*****.istio-system    SYNCED     SYNCED     SYNCED     NOT SENT     istiod-*****     1.12.1
istio-ingressgateway-*****.istio-system   SYNCED     SYNCED     SYNCED     NOT SENT     istiod-*****     1.12.1
foo-pod.default                           SYNCED     SYNCED     SYNCED     SYNCED       istiod-*****     1.12.1
bar-pod.default                           SYNCED     SYNCED     SYNCED     SYNCED       istiod-*****     1.12.1
baz-pod.default                           SYNCED     SYNCED     SYNCED     SYNCED       istiod-*****     1.12.1
```

<br>

### upgrade

#### ▼ upgradeとは

Istioのインプレースデプロイメントを実行する。

ℹ️ 参考：https://istio.io/latest/docs/setup/upgrade/in-place/

```bash
$ istioctl upgrade

This will install the Istio <バージョンタグ> default profile with ["Istio core" "Istiod" "Ingress gateways"] components into the cluster. Proceed? (y/N) y

✔ Istio core installed                                                                                                                                                                                       
✔ Istiod installed                                                                                                                                                                                           
✔ Ingress gateways installed                                                                                                                                                                                 
✔ Installation complete                                                                                                                                                                                      Making this installation the default for injection and validation.
```

<br>

### verify-install

#### ▼ verify-installとは

Istioリソースのapplyが正しく実行されたかを検証する。

ℹ️ 参考：https://istio.io/latest/docs/reference/commands/istioctl/#istioctl-verify-install

```bash
$ istioctl verify-install

1 Istio control planes detected, checking --revision "default" only
✔ ClusterRole: istiod-istio-system.istio-system checked successfully
✔ ClusterRole: istio-reader-istio-system.istio-system checked successfully

# 〜 中略 〜

✔ Service: istio-egressgateway.istio-system checked successfully
✔ ServiceAccount: istio-egressgateway-service-account.istio-system checked successfully
Checked 14 custom resource definitions
Checked 3 Istio Deployments
✔ Istio is installed and verified successfully
```

<br>

### version

#### ▼ versionとは

Istiodのバージョンを取得する。

```bash
$ istioctl version

client version: 1.12.1
pilot version: 1.12.1
pilot version: 1.7.2
data plane version: 1.12.1 (5 proxies)
```

<br>
