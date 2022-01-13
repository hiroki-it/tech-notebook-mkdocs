# Infrastructure as Code

## はじめに

本サイトにつきまして、以下をご認識のほど宜しくお願いいたします。

参考：https://hiroki-it.github.io/tech-notebook-mkdocs/md/

<br>

## 01. IaC

### IaCとは

構成ファイルの実装に基づくプロビジョニングによって、物理インフラや仮想インフラの構成を管理する手法のこと。

https://en.wikipedia.org/wiki/Infrastructure_as_code

<br>

### メリット

#### ・変更をコードレビュー可能

画面上からインフラを変更する場合、画面共有しながら操作し、レビューと変更を同時に行うことになる。コード化により、レビューを事前に行ったうえで変更する、という手順を踏める。

#### ・ヒューマンエラーが減る

画面上からの変更であると、ヒューマンエラーが起こってしまう。コード化すれば、これが減る。

#### ・再利用や冗長化が簡単

複数のアプリケーションのために、同様の設定で同様のインフラを構築する場合や、1つのアプリケーションのために、インフラを冗長化する場合、いくつも手動で構築する必要があり、労力がかかる。コード化すれば、これが楽になる。

#### ・過去の変更が記録に残る

画面上からの変更であると、過去の変更履歴が残らない。ソースコードをバージョン管理すれば、Issueと紐付けて、履歴を残せる。

<br>

### デメリット

#### ・運用のスピードが落ちる

運用時に、画面上からの操作であればすぐに終わる変更であるのにもかかわらず、コード化により、変更までに時間がかかる。そのため例えばAWSとすると、運用時に変更する頻度が多いインフラ（例：API Gateway（VPCリンクを含む）、IAMユーザ（紐付けるロールやポリシーを含む））はコード化せず、あえて画面上から構築する。

#### ・プロバイダーの機能追加に追従しにくい

プロバイダーは日々機能を追加している。画面上からの操作であればすぐにこの機能を支えるが、コード化により、ツールのバージョンをアップグレードしなければ、この機能を使えない。運用時に便利な機能をすぐに使えないため、インフラを改善できないことに繋がる。また、エンジニアの『新機能を使ってみたい欲』が行き場を失う。

#### ・リリースの心理的ハードルが高い

画面上から変更すれば、インフラ変更のリリース中に予期せぬエラーが起こることはまずない。しかし、コード化ツールでは、変更のリリース中に予期せぬエラーが起こる可能性は決して低くないため、リリースの心理的ハードルが高くなる。

<br>

## 02. プロビジョニング

### サーバープロビジョニング

#### ・サーバープロビジョニングとは

物理/仮想サーバーを最終的な状態に至らせるまでの一連の処理のこと。

参考：

- https://securesamba.com/term/%E3%83%97%E3%83%AD%E3%83%93%E3%82%B8%E3%83%A7%E3%83%8B%E3%83%B3%E3%82%B0/
- https://www.redhat.com/ja/topics/automation/what-is-provisioning#%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC%E3%83%97%E3%83%AD%E3%83%93%E3%82%B8%E3%83%A7%E3%83%8B%E3%83%B3%E3%82%B0

<br>

### コンテナプロビジョニング

#### ・コンテナプロビジョニングとは

コンテナを最終的な状態に至らせるまでの一連の作業のこと。

<br>

## 03. 命令型

### 命令型とは

インフラの構成順序を定義することによって、インフラを構築/更新/削除する手法のこと。インフラの操作順序を人間が理解しておく必要があり、インフラの構成管理のコストが高い。一方で、順番さえ理解していれば、構成ファイルを簡単に実装できるため、学習コストが低い。

参考：

- https://ja.wikipedia.org/wiki/Infrastructure_as_Code
- https://techblog.locoguide.co.jp/entry/2021/05/24/145342

<br>

### ツールの種類

#### ・物理/仮想サーバー（仮想マシン）

- Ansible
- Chef

#### ・コンテナ

- Ansible Container
- Dockerfile

#### ・クラウドインフラストラクチャ

- Ansible

<br>

## 04. 宣言型

### 宣言型とは

インフラのあるべき状態を定義することによって、インフラを構築/更新/削除する手法のこと。ツールごとに独自の宣言方法を持っており、学習コストが高い。その一方で、最終的な状態を定義しさえすれば、構築/更新/削除の順序はツールが解決してくれるため、インフラの構成管理のコストが少ない。

参考：

- https://ja.wikipedia.org/wiki/Infrastructure_as_Code
- https://techblog.locoguide.co.jp/entry/2021/05/24/145342

<br>

### ツールの種類

#### ・物理/仮想サーバー（仮想マシン）

- CFEngine
- Puppet
- Vagrantfile

#### ・コンテナオーケストレーション

- Docker compose
- Kubernetes

#### ・クラウドインフラストラクチャ

- AWS CloudFormation
- Azure Resource Manager
- GCP Deployment Manager
- SAM
- Serverless Framework
- Terraform