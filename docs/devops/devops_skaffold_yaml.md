---
title: 【IT技術の知見】skaffold.yaml＠Skaffold
description: skaffold.yaml＠Skaffoldの知見を記録しています。
---

# skaffold.yaml＠Skaffold

## はじめに

本サイトにつきまして、以下をご認識のほど宜しくお願いいたします。

ℹ️ 参考：https://hiroki-it.github.io/tech-notebook-mkdocs/about.html

<br>

## 01 Skaffoldの仕組み

### アーキテクチャ

調査中...

<br>

### パイプライン構成

#### ▼ ステージの種類

CI/CDのステージを持つ。build/test/deployステージに加えて、継続的な開発に役立つステージを持つ。ただし、Skaffoldは、基本的には開発環境でしか使わないため、ユースケースが限定的なステージもある。

![skaffold-pipeline](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/skaffold-pipeline.png)

#### ▼ 各ステージのツールの選択

Skaffoldの各ステージでは、それ専用のツールをコールできる。

ℹ️ 参考：https://skaffold.dev/docs/#skaffold-workflow-and-architecture

![skaffold-pipeline_tools](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/skaffold-pipeline_tools.png)

<br>

## 02. buildステージ

### buildステージとは

コンテナイメージのビルド方法を定義する。

ℹ️ 参考：https://skaffold.dev/docs/pipeline-stages/builders/

<br>

### artifacts

#### ▼ image

ビルドされるイメージの名前を設定する。

ℹ️ 参考：https://skaffold.dev/docs/references/yaml/#build-artifacts-image

```yaml
build:
  artifacts:
    - image: foo-app
    - image: foo-web
    - image: bar-app
    - image: bar-web
```

#### ▼ context

マイクロサービスのルートまでのパスを設定する。

ℹ️ 参考：https://skaffold.dev/docs/references/yaml/#build-artifacts-context

```yaml
build:
  artifacts:
    - image: foo-app
      context: ./src/foo
    - image: foo-web
      context: ./src/foo
    - image: bar-app
      context: ./src/bar
    - image: bar-web
      context: ./src/bar    
```

<br>

#### ▼ docker

| 項目         | 説明                                                         | 補足                                                         |
|------------| ------------------------------------------------------------ | ------------------------------------------------------------ |
| dockerfile | マイクロサービスのルートからDockerfileまでのパスを設定する。 | https://skaffold.dev/docs/references/yaml/#build-artifacts-docker-dockerfile |
| target     | ビルドするイメージのステージを設定する。                     | https://skaffold.dev/docs/references/yaml/#build-artifacts-docker-target |

```yaml
build:
  artifacts:
    - image: foo-app
      context: ./src/foo
      docker:
        dockerfile: ./docker/app/Dockerfile
    - image: foo-web
      context: ./src/foo
      docker:
        dockerfile: ./docker/web/Dockerfile
        target: development
    - image: bar-app
      context: ./src/bar
      docker:
        dockerfile: ./docker/app/Dockerfile
    - image: bar-web
      context: ./src/bar
      docker:
        dockerfile: ./docker/web/Dockerfile
        target: development
```

<br>

### local

#### ▼ push

ビルドしたコンテナイメージをリポジトリにプッシュするか否かを設定する。Skaffoldは基本的には開発環境でしか使わないため、これは無効化しておいた方が良い。

```yaml
build:
  local:
    push: false
```

#### ▼ useBuildkit

BuildKit機能の有効化を設定する。BuildKitではイメージレイヤーが並列的に作成されるため、ビルド時間を従来よりも短縮できる。

ℹ️ 参考：https://genzouw.com/entry/2021/07/17/100615/2724/

```yaml
build:
  local:
    useBuildkit: false
```

<br>

### tagPolicy

#### ▼ gitCommit

コミットIDをバージョンタグとして設定する。

ℹ️ 参考：https://skaffold.dev/docs/pipeline-stages/taggers/#gitcommit-uses-git-commitsreferences-as-tags

```yaml
build:
  tagPolicy:
    gitCommit: {}
```

#### ▼ sha256

sha256ハッシュ値と```latest```タグをバージョンタグとして設定する。

ℹ️ 参考：https://skaffold.dev/docs/pipeline-stages/taggers/#sha256-uses-latest-to-tag-images

```yaml
build:
  tagPolicy:
    sha256: {}
```

<br>

## 03. portForwardステージ

```skaffold run```コマンド時に、同時にポートフォワーディングを実行する。すでにポート番号が使用中だった場合は、```+1```されたポート番号が自動的に使用される。

```yaml
portForward:
  - resourceType: pod
    resourceName: foo-mysql-pod-0
    localPort: 3308 
    port: 3306
  - resourceType: pod
    resourceName: bar-mysql-pod-0  
    localPort: 3309
    port: 3306
```

<br>

## 04. testステージ

### testステージとは

Kubernetesリソースのテスト方法を定義する。

ℹ️ 参考：https://skaffold.dev/docs/pipeline-stages/testers/

<br>

### structureTests

ファイルを指定し、コンテナ構造テストを実施する。

ℹ️ 参考：https://qiita.com/Kta-M/items/83db480075caabcb0b7a

```yaml
test:
  - image: <イメージリポジトリURL> # <アカウントID>.dkr.ecr.ap-northeast-1.amazonaws.com/<イメージリポジトリ名>:latest
    structureTests:
      - ./structure-tests/foo.yaml
```

```yaml
schemaVersion: 2.0.0

# イメージにファイルが存在するか否かをテストする。
fileExistenceTests:
  - name: PHP file
    path: /var/www/public/index.php
    shouldExist: true
```

<br>

## 05. deployステージ

### deployステージとは

Kubernetesリソースのデプロイ手法を定義する。

ℹ️ 参考：https://skaffold.dev/docs/pipeline-stages/deployers/

<br>

### kubectl

#### ▼ kubectlとは

```kubectl```コマンドを使用して、Kubernetesリソースをデプロイする。ワイルドカードを使用できる。

ℹ️ 参考：https://skaffold.dev/docs/pipeline-stages/deployers/kubectl/

```yaml
deploy:
  kubectl:
    manifests:
      - ./release/dev/kubernetes.yaml
      - ./**/**/**.yaml # ワイルドカードを使用できる。
```

<br>

### helm

#### ▼ helm

Helmを使用して、Kubernetesリソースをデプロイする。

ℹ️ 参考：https://skaffold.dev/docs/pipeline-stages/deployers/helm/

```yaml
deploy:
  helm:
    releases:
    - name: <リリース名>
      artifactOverrides:
        image: <コンテナイメージ名> # buildステージのartifactsのコンテナイメージ名と合わせる。
      imageStrategy:
        helm: {}
```

<br>
