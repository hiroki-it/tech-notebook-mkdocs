---
title: 【IT技術の知見】Istio＠仮想化
description: Istio＠仮想化の知見を記録しています。
---

# Istio＠仮想化

## はじめに

本サイトにつきまして、以下をご認識のほど宜しくお願いいたします。

ℹ️ 参考：https://hiroki-it.github.io/tech-notebook-mkdocs/about.html

<br>

## 01. Istioの仕組み

### アーキテクチャ

データプレーンとコントロールプレーンによって、サービスメッシュを実現する。マイクロサービス間の通信を透過的にする（通信の存在を感じさせない）ことを思想としている。ただ必ずしも、Istioリソースを使用する必要はなく、KubernetesやOpenShiftに内蔵されたIstioに相当する機能を使用しても良い。

ℹ️ 参考：

- https://istio.io/latest/docs/ops/deployment/architecture/
- https://techblog.zozo.com/entry/zozotown-istio-production-ready

![istio_architecture](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/istio_architecture.png)

<br>

### データプレーン

#### ▼ データプレーンとは

インバウンド通信をマイクロサービスにルーティングする機能を持つ。Istioは、プロキシ機能を持つistio-proxyコンテナを自動的に作成し、これがマイクロサービスに通信をルーティングする。

ℹ️ 参考：https://www.tigera.io/blog/running-istio-on-kubernetes-in-production-part-i/

#### ▼ Proxy

| コンテナ名        | 機能                                                                                                            |
| ----------------- |---------------------------------------------------------------------------------------------------------------|
| ```istio-proxy``` | Envoyが稼働しており、VirtualServiceとDestinationRuleの設定値はenvoyの構成情報としてコンテナに適用される。<br>ℹ️ 参考：https://sreake.com/blog/istio/ |
| ```istio-init```  | iptablesのルールをPodに適用する。これにより、Podは受信したいずれのインバウンド通信を```istio-proxy```コンテナにルーティングするか、を決定する。                       |

#### ▼ ```istio-proxy```コンテナが作成される仕組み

サイドカーコンテナのistio-proxyコンテナをPod内に自動的に作成する処理は、admission-controllersのmutating-admission-webhookを使用した機能である。istio-injectionが有効になっている場合、Podの作成処理時に、KubernetesからIstioにWebhookが送信される。具体的には、Pod、Deployment、StatefulSet、DaemonSet、によるPodの作成処理でkube-apiserverにコールすると、mutating-admission時に、WebhookがIstio内のsidecar-injector-webhookサーバーの```/inject```エンドポイントに送信される。これを受信したsidecar-injector-webhookサーバーは、istio-proxyコンテナを作成する処理を返信する。Kubernetesはこれを受信し、Podにistio-proxyコンテナを作成する。

ℹ️ 参考：https://www.sobyte.net/post/2022-07/istio-sidecar-injection/

![kubernetes_admission-controllers_istio-injection.ong](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/kubernetes_admission-controllers_istio-injection.ong.png)


<br>

### コントロールプレーン

#### ▼ コントロールプレーンとは

データプレーンを包括的に管理する機能を持つ。Istioは、istio-proxyコンテナの管理機能を持つistidというPodを作成する。このPod内には、Pilot、Citadel、Galley、に相当するコンテナが稼働している。

ℹ️ 参考：

- https://project.nikkeibp.co.jp/idg/atcl/idg/17/020100207/020100001/?ST=idg-cm-network&P=2
- https://www.tigera.io/blog/running-istio-on-kubernetes-in-production-part-i/

#### ▼ Citadel

マイクロサービス間の認証やトレースIDを管理する。

#### ▼ Galley

コンテナオーケストレーションツール（Kubernetes、OpenShift、など）の種類を認識し、ツールに合ったIstiodコンポーネントを作成する。

#### ▼ Pilot

コンテナオーケストレーションツール（Kubernetes、OpenShift、など）の種類を認識し、ツールに合ったプロキシコンテナを作成する。他に、Istioの設定を、Istioによって注入されるEnvoyの設定に変換する。

ℹ️ 参考：https://blog.devgenius.io/implementing-service-discovery-for-microservices-df737e012bc2

| コンテナ名      | 機能                                                         |
| --------------- | ------------------------------------------------------------ |
| ```discovery``` | サービスレジストリに登録された情報を基に、マイクロサービスが他のマイクロサービスを識別する。（サービスディスカバリー） |
| ```agent```     | istio-proxyコンテナを起動する。                              |

#### ▼ Mixer

v1.5からデータプレーン側に統合された。

ℹ️ 参考：https://www.elastic.co/jp/blog/istio-monitoring-with-elastic-observability

<br>

### Istio、Envoy（Istio無し）、Kubernetesの対応関係

Kubernetes、Envoy、Kubernetesの比較は以下の通り

ℹ️ 参考：

- https://thenewstack.io/why-do-you-need-istio-when-you-already-have-kubernetes/
- https://www.mirantis.com/blog/your-app-deserves-more-than-kubernetes-ingress-kubernetes-ingress-vs-istio-gateway-webinar/
- https://github.com/envoyproxy/go-control-plane
- https://istiobyexample-ja.github.io/istiobyexample/ingress/

| Istio＋Kubernetes＋Envoy | Kubernetes＋Envoy | Kubernetesのみ                 |
| ------------------------ | ----------------- | ------------------------------ |
| DestinationRule          | Route             | kube-proxy                     |
| EnvoyFilter              | Listener          | kube-proxy                     |
| Istiod                   | go-control-plane  | -                              |
| ServiceEntry             | Cluster           | Service                        |
| VirtualService＋Gateway  | Route＋Listener   | Ingress＋Ingressコントローラー |
| WorkloadEntry            | Endpoint          | Endpoint                       |

<br>

### IaC

IstioのIaCについては、以下のリンクを参考にせよ。

ℹ️ 参考：https://hiroki-it.github.io/tech-notebook-mkdocs/infrastructure_as_code/infrastructure_as_code_container_kubernetes_custom_resource_istio_manifest_file.html

<br>

## 02. リソースとオブジェクト

### Istioリソース

Istioの各コンポーネントのことにより、Kubernetesのカスタムリソースとして定義されている。Istioリソースは、IaCによってマニフェストファイルで定義される。Istioのマニフェストファイルについては、以下のリンクを参考にせよ。

ℹ️ 参考：https://hiroki-it.github.io/tech-notebook-mkdocs/infrastructure_as_code/infrastructure_as_code_container_kubernetes_custom_resource_istio_manifest_file.html

<br>

### Istioオブジェクト

マニフェストファイルによって量産されたIstioリソースのインスタンスのこと。

<br>

## 02-02. インバウンド通信に関するリソース

### IngressGateway

#### ▼ IngressGatewayとは

![istio_ingress-gateway](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/istio_ingress-gateway.png)

Gateway、VirtualService、DestinationRuleの設定を基に、Clusterネットワーク外からインバウンド通信を受信し、Podにルーティングする。```istio-ingressgateway```というLoadBalancer Serviceと、```istio-ingressgateway-*****```というPodから構成される

ℹ️ 参考：

- https://istio.io/latest/docs/tasks/traffic-management/ingress/ingress-control/
- https://qiita.com/kenyashiro/items/b94197890de434ed9ceb
- https://qiita.com/J_Shell/items/296cd00569b0c7692be7
- https://blog.jayway.com/2018/10/22/understanding-istio-ingress-gateway-in-kubernetes/

<br>

### Gateway

#### ▼ Gatewayとは

IngressGatewayの機能のうち、Clusterネットワーク外から受信したインバウンド通信をフィルタリングする機能を担う。

ℹ️ 参考：https://istio.io/latest/blog/2018/v1alpha3-routing/

<br>

### VirtualService

#### ▼ VirtualServiceとは

IngressGatewayの機能のうち、IngressGatewayで受信したインバウンド通信をいずれのServiceにルーティングするか、を決定する機能を担う。Service自体の設定は、IstioではなくKubernetesで行うことに注意する。ルーティング先のServiceが見つからないと、```404```ステータスを返信する。

ℹ️ 参考：

- https://tech.uzabase.com/entry/2018/11/26/110407
- https://knowledge.sakura.ad.jp/20489/

#### ▼ Envoyの設定値として

VirtualServiceの設定値は、Envoyのフロントプロキシの設定値としてIstioリソースに適用される。

ℹ️ 参考：

- https://istio.io/latest/docs/concepts/traffic-management/
- http://blog.fujimisakari.com/service_mesh_and_routing_and_lb/
- https://sreake.com/blog/istio/

#### ▼ VirtualService数

ℹ️ 参考：https://www.moesif.com/blog/technical/api-gateways/How-to-Choose-The-Right-API-Gateway-For-Your-Platform-Comparison-Of-Kong-Tyk-Apigee-And-Alternatives/ 

| 場合                                 | VirtualService数                                             |
| ------------------------------------ | ------------------------------------------------------------ |
| API GatewayをIstio内で管理する場合   | 外部からのインバウンド通信をAPI GatewayにルーティングするVirtualServiceを1つだけ作成しておけばよい。 |
| API GatewayをIstio内で管理しない場合 | API Gatewayから全てのマイクロサービスにルーティングできるように、各マイクロサービスにルーティングできるVirtualServiceを定義する必要がある。 |

<br>

## 02-03. アウトバウンド通信に関するリソース

### EgressGateway

#### ▼ EgressGatewayとは

Clusterネットワーク内からアウトバウンド通信を受信し、フィルタリングした後、パブリックネットワークにルーティングする。

ℹ️ 参考：https://knowledge.sakura.ad.jp/20489/

![istio_gateway](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/istio_gateway.png)

<br>

### ServiceEntry

#### ▼ ServiceEntryとは

アウトバウンド通信のうち、送信できるもののみを指定したドメインやEgressGatewayにルーティングする。ServiceEntryを使用しない場合は、全てのアウトバウンド通信がルーティングされる。

ℹ️ 参考：https://tech.uzabase.com/entry/2018/11/26/110407

![istio_service-entry](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/istio_service-entry.png)

<br>

## 02-04. 両方向通信に関するリソース

### DestinationRule

#### ▼ DestinationRuleとは

| 通信方向       | 機能                                                         | 補足                                                         |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| インバウンド   | IngressGatewayの機能のうち、Serviceで受信したインバウンド通信をいずれのPodにルーティングするか、を決定する機能を担う。Service自体の設定は、IstioではなくKubernetesで行うことに注意する。 |                                                              |
| アウトバウンド | istio-proxyコンテナの送信するアウトバウンド通信をTLSで暗号化するか否か、を決定する機能を担う。 | ℹ️ 参考：https://istio.io/latest/docs/ops/configuration/traffic-management/tls-configuration/#sidecars |

#### ▼ Envoyの設定値として

DestinationRuleの設定値は、Envoyのリバースプロキシコンテナの設定値としてistio-proxyコンテナに適用される。

ℹ️ 参考：

- https://istio.io/latest/docs/concepts/traffic-management/
- http://blog.fujimisakari.com/service_mesh_and_routing_and_lb/
- https://sreake.com/blog/istio/

<br>

## 03. Istiod

### Istiodとは

istio-proxyコンテナを統括的に管理する。

ℹ️ 参考：

- https://istio.io/latest/docs/ops/deployment/architecture/
- https://speakerdeck.com/kurochan/ru-men-envoy?slide=34

<br>

### Citadal

#### ▼ Citadalとは

暗号鍵やSSL証明書を管理する。

ℹ️ 参考：https://knowledge.sakura.ad.jp/20489/

<br>

### Galley

#### ▼ Galleyとは

<br>

### sidecar-injector

#### ▼ sidecar-injectorとは

istio-proxyコンテナをサイドカーとして稼働させる。

<br>

### Mixer

#### ▼ Mixerとは

認証やデータ収集を行う。

<br>

### Pilot

#### ▼ Pilotとは

Serviceディスカバリやトラフィックの管理を行う。

<br>

## 04. IstioOperator

### IstioOperatorとは

Istioのインストールや、Istioリソースの操作ができるリソースのこと。

ℹ️ 参考：

- https://istio.io/latest/docs/reference/config/istio.operator.v1alpha1/
- https://istio.io/latest/docs/setup/install/operator/#update

<br>

## 05. Injectionテスト

### Faultインジェクション

#### ▼ Faultインジェクションとは

障害を意図的に注入し、サービスメッシュの動作を検証する。

ℹ️ 参考：https://istio.io/latest/docs/tasks/traffic-management/fault-injection/

#### ▼ テストの種類

| テスト名         | 内容                                                         |
| ---------------- | ------------------------------------------------------------ |
| Deplayインジェクション | マイクロサービスに対するインバウンド通信にて、意図的に通信の遅延を引き起こす。<br>ℹ️ 参考：https://istio.io/latest/docs/tasks/traffic-management/fault-injection/#injecting-an-http-delay-fault |
| Abortインジェクション  | マイクロサービスに対するインバウンド通信にて、意図的に通信の中止を引き起こす。<br>ℹ️ 参考：https://istio.io/latest/docs/tasks/traffic-management/fault-injection/#injecting-an-http-abort-fault |

<br>

## 06. 障害対策

### サーキットブレイカー

#### ▼ サーキットブレイカーとは

マイクロサービス間に設置され、他のマイクロサービスに連鎖的に起こる障害（カスケード障害）を吸収する仕組みのこと。爆発半径を最小限にできる。下流マイクロサービスに障害が発生した時に、上流マイクロサービスにエラーを返してしまわないよう、一旦マイクロサービスへのルーティングを停止し、直近の成功時の処理結果を返信する。

ℹ️ 参考：https://digitalvarys.com/what-is-circuit-breaker-design-pattern/

![circuit-breaker](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/circuit-breaker.png)

<br>

## 07. 認証

マイクロサービスアーキテクチャにおける認証にはいくつか種類がある。そのうち、Istioは『分散型』と『ゲートウェイ分散型』の認証を実現することを助ける。

ℹ️ 参考：

- https://istio.io/latest/docs/concepts/security/#authentication-architecture
- https://hiroki-it.github.io/tech-notebook-mkdocs/software/software_application_architecture_backend_microservices.html

<br>

## 07-02. 認可

ℹ️ 参考：https://istio.io/latest/docs/concepts/security/#authorization

