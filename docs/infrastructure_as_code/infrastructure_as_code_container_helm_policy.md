---
title: 【知見を記録するサイト】設計ポリシー＠Helm
description: 設計ポリシー＠Helmの知見をまとめました．
---

# 設計ポリシー＠Helm

## はじめに

本サイトにつきまして，以下をご認識のほど宜しくお願いいたします．

参考：https://hiroki-it.github.io/tech-notebook-mkdocs/about.html

<br>

## 01. ディレクトリ構成 

### chartディレクトリ

#### ・必須の要素

ルートディレクトリに```Chart.yaml```ファイルと```template```ディレクトリを置く必要がある．また，チャートのコントリビュート要件も参考にすること．

参考：

- https://github.com/helm/charts/blob/master/CONTRIBUTING.md#technical-requirements
- https://helm.sh/docs/topics/charts/#the-chart-file-structure
- https://mixi-developers.mixi.co.jp/argocd-with-helm-7ec01a325acb

```bash
repository/
├── chart/
│   ├── charts/ # 依存する他のチャートを配置する．
│   ├── temlaptes/ # ユーザー定義のチャートを配置する．ディレクトリ構造は自由である．
│   │   ├── tests/
│   │   ├── _helpers.tpl
│   │   └── template.yaml # チャートの共通ロジックを設定する．
│   │
│   ├── .helmignore # チャートアーカイブの作成時に無視するファイルを設定する．
│   ├── Chart.yaml # チャートの概要を設定する．
│   └── values.yaml # テンプレートの変数に出力する値を設定する．
...
```

#### ・稼働環境別

稼働環境別に```values.yaml```ファイルと```.tpl```ファイルを作成する．```.tpl```ファイルは```templates```ディレクトリ内に置く必要がある．テンプレートからマニフェストファイルを作成する時に，各環境の```values.yaml```を参照する．

参考：https://github.com/codefresh-contrib/helm-promotion-sample-app

```bash
repository/
├── chart/
│   ├── temlaptes/
│   │   ├── manifests/ # 共通のマニフェストファイル
│   │   ├── tpls/ # .tplファイル
│   │   │   ├── prd/
│   │   │   ├── stg/
│   │   │   └── dev/         
│   │   ...
│   │
│   ├── values
│   │   ├── prd.yaml # 本番環境へのデプロイ時に出力する値
│   │   ├── stg.yaml # ステージング環境 〃
│   │   └── dev.yaml # 開発環境 〃
...
```

<br>

## 02. 命名規則

### templateディレクトリ

#### ・命名規則

ファイル名はスネークケースとし，Kubernetesリソースを識別できる名前とする．

参考：https://helm.sh/docs/chart_best_practices/templates/

#### ・拡張子

拡張子は```.yaml```とする．

参考：https://helm.sh/docs/chart_best_practices/templates/

<br>