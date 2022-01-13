# kubectlコマンド

## はじめに

本サイトにつきまして、以下をご認識のほど宜しくお願いいたします。

参考：https://hiroki-it.github.io/tech-notebook-mkdocs/md/

<br>

## 01. コマンド

### apply

#### ・applyとは

同じ識別子（オブジェクト名）のオブジェクトが存在しない場合はオブジェクトを作成し、存在する場合はマニフェストファイルの差分を更新する。全ての項目を更新できるわけでない。

参考：https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply

**＊例＊**

マニフェストファイルを指定し、```apply```コマンドを実行する。

```bash
# オブジェクトを作成する。
$ kubectl apply -f ./kubernetes-manifests/<マニフェストファイル名>.yml

pod/foo-pod created
```

```bash
# 設定値を変更する。
$ kubectl apply -f ./kubernetes-manifests/<マニフェストファイル名>.yml

pod/foo-pod configured
```

<br>

### config

#### ・configとは

kubernetesコマンドに関するパラメータを操作する。

参考：https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#config

#### ・view

パラメータのデフォルト値が設定された```~/.kude/config```ファイルを表示する。

**＊例＊**

```bash
$ kubectl config view

apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://kubernetes.docker.internal:6443
  name: docker-desktop
contexts:
- context:
    cluster: docker-desktop
    user: docker-desktop
  name: docker-desktop
current-context: docker-desktop
kind: Config
preferences: {}
users:
- name: docker-desktop
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

<br>

### cp

#### ・cpとは

ホストPCのファイルまたはディレクトリを指定したPod内のコンテナにコピーする。

参考：https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#cp

#### ・オプション無し

```bash
$ kubectl cp <ホストPCのファイルパス> <名前空間名>/<PodID>:<コンテナのファイルパス>
```

```bash
$ kubectl cp <ホストPCのファイルパス> <名前空間名>/<PodID>:<コンテナのディレクトリパス>/
```

<br>

### create

#### ・createとは

様々なオブジェクトを作成する。```expose```コマンドと```run```コマンドで作成できるオブジェクトを含む様々なものを作成できるが、オプションが少ない。そのため、```f```オプションでマニフェストファイルを指定し、おぶえジェクトを作成した方が良い。同じ識別子（オブジェクト名）のオブジェクトが存在する場合は重複エラーになる。

参考：https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create

**＊例＊**

マニフェストファイルを指定し、```create```コマンドを実行する。

```bash
$ kubectl create -f ./kubernetes-manifests/foo-pod.yml

pod/foo-pod created
```

```bash
$ kubectl create -f ./kubernetes-manifests/foo-service.yml

service/foo-service created
```

#### ・deployment

Pod数を維持管理するReplicaSetを作成する。Podを削除するためには、Deployment自体を削除しなければならない。

**＊例＊**

```bash
$ kubectl create deployment -f ./kubernetes-manifests/foo-deployment.yml
```

#### ・secret generic

Secretを作成する。

参考：

- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-secret-generic-em-
- https://qiita.com/toshihirock/items/38d09b2822a347c3f958

**＊例＊**

指定した```.env```ファイルからSecretを作成する。

```bash
$ kubectl create secret generic foo-secret --from-env-file=./foo/.env

secret/foo-secret created
```

指定した```.env```ファイル以外からSecretを作成する。

```bash
$ kubectl create secret generic foo-secret --from-file=./foo/values.txt

secret/foo-secret created
```

キー名と値からSecretを作成する。

```bash
$ kubectl create secret generic foo-secret --from-literal=username="test" --from-literal=password="test"

secret/foo-secret created
```

<br>

### exec

#### ・execとは

指定したPod内のコンテナでコマンドを実行する。

参考：https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#exec

#### ・-it

**＊例＊**

コンテナを指定して、デタッチモードで ```exec```コマンドを実行する。

```bash
$ kubectl exec -it <Pod名> -c <コンテナ名> -- bash

[root@<Pod名>] $ ls -la 
```

コンテナを指定しない場合は、デフォルトのコンテナが選ばれる。Podのラベル名ではなく、Pod名であることに注意する。

```bash
$ kubectl exec -it <Pod名> -- bash

Defaulted container "foo-container" out of: foo-container, bar-container
```

<br>

### expose

#### ・exposeとは

Serviceを作成する。

参考：

- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#expose
- https://qiita.com/sourjp/items/f0c8c8b4a2a494a80908

#### ・--type、--port、--target-port

**＊例＊**

ClusterIP Serviceを作成する。

```bash
$ kubectl expose <Service名> --type=ClusterIP --port=<受信ポート番号> --target-port=<転送先ポート番号>
```

NodePort Serviceを作成する。

```bash
$ kubectl expose <Service名> --type=NodePort --port=<受信ポート番号> --target-port=<転送先ポート番号>
```

LoadBalancer Serviceを作成する。

```bash
$ kubectl expose <Service名> --type=LoadBalancer --port=<受信ポート番号> --target-port=<転送先ポート番号>
```

<br>

### get <オブジェクト>

#### ・getとは

オブジェクトを参照する。

参考：https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get

**＊例＊**

指定したノードの情報を表示する。

```bash
$ kubectl get nodes 

NAME             STATUS   ROLES                  AGE   VERSION
docker-desktop   Ready    control-plane,master   12h   v1.21.5 # マスターノード
```

指定したPodの情報を表示する。

```bash
$ kubectl get pods

NAME       READY   STATUS             RESTARTS   AGE
foo-pod    0/2     ImagePullBackOff   0          7m52s
```

**＊例＊**

指定したServiceの情報を表示する。

```bash
$ kubectl get services

NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
foo-service    ClusterIP   nn.nnn.nnn.n   <none>        80/TCP    10s
kubernetes     ClusterIP   nn.nn.n.n      <none>        443/TCP   12h
```

#### ・-o

**＊例＊**

指定したSecretをYAML形式で表示する。

```bash
$ kubectl get secret <Secret名> -o yaml

apiVersion: v1
data:
  FOO: ***** # base64エンコード値
  BAR: *****
  BAZ: *****
kind: Secret
metadata:
  creationTimestamp: "2021-12-00T00:00:00Z"
  name: swp-secret
  namespace: default
  resourceVersion: "18329"
  uid: 507e3126-c03b-477d-9fbc-9434e7aa1920
type: Opaque
```

<br>

### label

#### ・labelとは

指定したオブジェクトのラベルを操作する。

#### ・オプション無し

**＊例＊**

指定したオブジェクトにラベルを作成する。

```bash
$ kubectl label <オブジェクト名> foo=bar
```

指定したオブジェクトのラベルを削除する。

```bash
$ kubectl label <オブジェクト名> foo-
```

#### ・--overwrite

**＊例＊**

指定したオブジェクトにラベルの値を変更する。

```bash
$ kubectl label --overwrite <オブジェクト名> foo=bar
```

<br>

### logs

#### ・logsとは

指定したオブジェクトのログを表示する。

参考：https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs

#### ・オプション無し

**＊例＊**

Pod名とコンテナ名を指定し、コンテナのログを表示する。

```bash
$ kubectl logs -n <名前空間名> <Pod名> -c <コンテナ名>

2021/11/27 08:34:01 [emerg] *****
```

名前空間、Pod名、コンテナ名を指定し、kube-proxyのログを確認する。

```bash
$ kubectl logs -n kube-system <Pod名> -c kube-proxy

I1211 05:34:22.262955       1 node.go:172] Successfully retrieved node IP: nnn.nnn.nn.n
I1211 05:34:22.263084       1 server_others.go:140] Detected node IP nnn.nnn.nn.n
W1211 05:34:22.263104       1 server_others.go:565] Unknown proxy mode "", assuming iptables proxy
I1211 05:34:22.285367       1 server_others.go:206] kube-proxy running in dual-stack mode, IPv4-primary
I1211 05:34:22.285462       1 server_others.go:212] Using iptables Proxier.
I1211 05:34:22.285484       1 server_others.go:219] creating dualStackProxier for iptables.
W1211 05:34:22.285508       1 server_others.go:495] detect-local-mode set to ClusterCIDR, but no IPv6 cluster CIDR defined, , defaulting to no-op detect-local for IPv6
I1211 05:34:22.286807       1 server.go:649] Version: v1.22.3
I1211 05:34:22.289459       1 config.go:315] Starting service config controller
I1211 05:34:22.289479       1 shared_informer.go:240] Waiting for caches to sync for service config
I1211 05:34:22.289506       1 config.go:224] Starting endpoint slice config controller
I1211 05:34:22.289525       1 shared_informer.go:240] Waiting for caches to sync for endpoint slice config
I1211 05:34:22.389800       1 shared_informer.go:247] Caches are synced for endpoint slice config 
I1211 05:34:22.389956       1 shared_informer.go:247] Caches are synced for service config 
```

<br>

### patch

#### ・patchとは

JSON/YAML形式を入力値として、オブジェクトの設定を変更する。

参考：https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#patch

**＊例＊**

削除されないボリュームを削除する。

参考：https://github.com/kubernetes/kubernetes/issues/77258#issuecomment-514543465

```bash
$ kubectl get pv \
  | tail -n+2 \
  | awk '{print $1}' \
  | xargs -I{} kubectl patch pv {} -p '{"metadata":{"finalizers": null}}'
```

<br>

### port-forward

#### ・port-forwardとは

ホストのポートから指定したオブジェクトのポートに対して、ポートフォワーディングを実行する。開発環境にて、Serviceを経由せずに直接Podにリクエストを送信したい場合や、SQLクライアントを用いてPod内のDBコンテナにTCP/IP接続したい場合に使用する。

参考：

- https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/#forward-a-local-port-to-a-port-on-the-pod
- https://stackoverflow.com/questions/53898627/mysql-remote-connect-over-ssh-to-a-kubernetes-pod

**＊例＊**

```bash
$ kubectl port-forward <Pod名> <ホストポート>:<Podポート>
```

<br>

### proxy

#### ・proxyとは

ローカルホストとkube-apiserverの間にプロキシとして機能するオブジェクトを作成する。kube-proxyとは異なるオブジェクトであることに注意する。

#### ・--address、--accept-hosts

参考：https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#proxy

**＊例＊**

```bash
$ kubectl proxy --address=0.0.0.0 --accept-hosts='.*'  

Starting to serve on [::]:8001
```

<br>

### run

#### ・runとは

Deployment、Pod、ジョブを作成する。

参考：https://qiita.com/sourjp/items/f0c8c8b4a2a494a80908

#### ・--restart、--image、--port

**＊例＊**

もし```restart```オプションが```Always```なら、Deploymentが作成される。


```bash
$ kubectl run <Deployment名> --restart=Always --image=<イメージ名>:<タグ名> --port=<ポート番号>
```

もし```restart```オプションが```Never```なら、Podが作成される。

```bash
$ kubectl run <Pod名> --restart=Never --image=<イメージ名>:<タグ名> --port=<ポート番号>
```

もし```restart```オプションが```OnFailure```なら、ジョブが作成される。

```bash
$ kubectl run <ジョブ名> --restart=OnFailure --image=<イメージ名>:<タグ名> --port=<ポート番号>
```
