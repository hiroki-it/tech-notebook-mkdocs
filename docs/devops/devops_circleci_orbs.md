---
title: 【知見を記録するサイト】CircleCI＠Orbs
description: CircleCI＠Orbsの知見をまとめました．
---

# CircleCI＠Orbs

## はじめに

本サイトにつきまして，以下をご認識のほど宜しくお願いいたします．

参考：https://hiroki-it.github.io/tech-notebook-mkdocs/about.html

<br>

## 01. Orbs

### Orbsとは

CircleCIから提供される汎用的なパッケージの使用を読み込む．

<br>

### 構成

| オプション名 | 説明                                  |
| ------------ | ------------------------------------- |
| jobs         | ```job```キーに割り当てられる．       |
| commands     | ```step```キーに割り当てられる．      |
| executors    | ```executors```キーに割り当てられる． |

<br>

## 01-02. セットアップ

### インストール

**＊実装例＊**

```yaml
version: 2.1

orbs:
    hello: circleci/hello-build@0.0.5
    
workflows:
    "Hello Workflow":
        jobs:
          - hello/hello-build
```

<br>

#### ・Orbsのデメリット

Orbsのパッケージの処理の最小単位は```step```である．そのため，```step```よりも小さい```run```はOrbsに組み込むことができず，```run```固有のオプションや```run```に設定できるlinuxコマンドをOrbsでは使用できないことになる．

#### ・オプションへの引数の渡し方と注意点

AWS認証情報は，CircleCIのデフォルト名と同じ環境変数名で登録しておけば，オプションで渡さなくとも，自動で入力してくれる．オプションが```env_var_name```型は，基本的に全てのスコープレベルの環境変数を受け付ける．ただしAlpine Linuxでは，『```$BASH_ENV```』を用いて，複数の```run```間で環境変数を共有できず，orbsのステップに環境変数を渡せないため注意する．

参考：https://github.com/circleci/circleci-docs/issues/1650

**＊実装例＊**

```yaml
version: 2.1

orbs:
  aws-foo: circleci/aws-foo@x.y.z

jobs:
  foo_bar_baz:
    docker:
      - image: circleci/python:x.y.z
    steps:
      - attach_workspace:
          at: .
      - setup_remote_docker:
      - aws-cli/install
      - aws-cli/setup
      - aws-foo/foo-bar-baz:
          # デフォルト名であれば，記述しなくても自動的に入力してくれる．
          account-url: $AWS_ECR_ACCOUNT_URL_ENV_VAR_NAME
          aws-access-key-id: $ACCESS_KEY_ID_ENV_VAR_NAME
          aws-secret-access-key: $SECRET_ACCESS_KEY_ENV_VAR_NAME
          region: $AWS_REGION_ENV_VAR_NAME
```

<br>

## 02. aws-cli

### commands

#### ・install

aws-cliコマンドのインストールを行う．

#### ・setup

aws-cliコマンドのインストールと，Credentials情報の設定を行う．AWSリソースを操作するために用いる．

**＊実装例＊**

CloudFrontに保存されているキャッシュを削除する．フロントエンドをデプロイしたとしても，CloudFrontに保存されているキャッシュを削除しない限り，キャッシュがHitしたユーザーには過去のファイルがレスポンスされてしまう．そのため，S3へのデプロイ後に，キャッシュを削除する必要がある．

```yaml
version: 2.1

orbs:
  aws-cli: circleci/aws-cli@1.3.1

jobs:
  cloudfront_create_invalidation:
    docker:
      - image: cimg/python:3.9-node
    steps:
      - checkout
      - aws-cli/setup
      - run:
          name: Run create invalidation
          command: |
            echo $AWS_CLOUDFRONT_ID |
            base64 --decode |
            aws cloudfront create-invalidation --distribution-id $AWS_CLOUDFRONT_ID --paths "/*"
            
workflows:
  # ステージング環境にデプロイ
  develop:
    jobs:
      # 直前に承認ジョブを挿入する
      - hold:
          name: hold_create_invalidation_stg
          type: approval
      - cloudfront_create_invalidation:
          name: cloudfront_create_invalidation_stg
          filters:
            branches:
              only:
                - develop
                
  # 本番環境にデプロイ                
  main:
    jobs:
      # 直前に承認ジョブを挿入する
      - hold:
          name: hold_create_invalidation_prd
          type: approval    
      - cloudfront_create_invalidation:
          name: cloudfront_create_invalidation_prd
          filters:
            branches:
              only:
                - main   
```

ただし，```credentials```ファイルの作成では，orbsを用いない方がより簡潔に条件分岐を実装できるかもしれない．

```bash
#!/bin/bash

set -xeuo pipefail

case "$APP_ENV" in
    "stg")
        AWS_ACCESS_KEY_ID=$STG_AWS_ACCESS_KEY_ID
        AWS_SECRET_ACCESS_KEY=$STG_AWS_SECRET_ACCESS_KEY
    ;;
    "prd")
        AWS_ACCESS_KEY_ID="$PRD_AWS_ACCESS_KEY_ID"
        AWS_SECRET_ACCESS_KEY="$PRD_AWS_SECRET_ACCESS_KEY"
    ;;
    *)
        echo "The parameter ${APP_ENV} is invalid."
        exit 1
    ;;
esac

# defaultプロファイルにクレデンシャル情報を設定する．
aws configure << EOF
$(echo $AWS_ACCESS_KEY_ID)
$(echo $AWS_SECRET_ACCESS_KEY)
$(echo $AWS_DEFAULT_REGION)
json
EOF

# 正しく設定されたかを確認する．
aws configure list
```

<br>

## 03. aws-ecr

### jobs

#### ・build-and-push-image

CircleCIコンテナでdockerイメージをビルドし，ECRにデプロイする．```remote-docker-layer-caching```を用いて，Docker Layer Cacheを有効化できる．

**＊実装例＊**

```yaml
version: 2.1

orbs:
  aws-cli: circleci/aws-cli@1.3.1
  aws-ecr: circleci/aws-ecr@6.15.2

jobs:
  aws-ecr/build-and-push-image:
    name: ecr_build_and_push_image
    # Docker Layer Cacheを用いるかどうか（有料）
    remote-docker-layer-caching: true
    # リポジトリがない時に作成するかどうか．
    create-repo: true
    no-output-timeout: 20m
    # projectを作業ディレクトリとした時の相対パス
    dockerfile: ./infra/docker/Dockerfile
    path: "."
    profile-name: myProfileName
    repo: "{$SERVICE}-repository"
    # CircleCIのハッシュ値によるバージョニング
    tag: $CIRCLE_SHA1
    # job内にて，attach_workspaceステップを実行．
    attach-workspace: true
    # attach_workspaceステップ実行時のrootディレクトリ
    workspace-root: <ディレクトリ名>
```

<br>

## 04. aws-ecs

### jobs

#### ・deploy-update-service（ローリングアップデート使用時）

ECRイメージを用いて，新しいリビジョン番号のタスク定義を作成し，またこれを用いてコンテナをデプロイする．

| 設定値                             | 説明                                                         |                                                              |
| ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ```container-image-name-updates``` | コンテナ定義のコンテナ名とイメージタグを上書きする．         | イメージはCircleCIのハッシュ値でタグ付けしているので必須．   |
| ``` verify-revision-is-deployed``` | ローリングアップデートのタスクがタスク定義のタスク必要数に合致したかを継続的に監視する． | 例えば，タスクが『Runnning』にならずに『Stopped』になってしまう場合や，既存のタスクが『Stopped』にならずに『Running』のままになってしまう場合，この状態はタスクの必要数に合致しないので，検知できる． |
| ```max-poll-attempts```            | ポーリングの最大試行回数を設定する．```poll-interval```と掛け合わせて，そう実行時間を定義できる． | 総実行時間を延長する時，間隔秒数はできるだけ短い方が無駄な実行時間が発生しないため，最大回数を増やす． |
| ```poll-interval```                | 試行の間隔秒数を設定する．```max-poll-attempts```と掛け合わせて，そう実行時間を定義できる． |                                                              |

オプションを用いて，```max-poll-attempts```（ポーリングの最大試行回数）と```poll-interval```（試行の間隔秒数）で，ポーリングの総実行時間を定義できる．

参考：https://circleci.com/docs/ja/2.0/ecs-ecr/#deploy-the-new-docker-image-to-an-existing-aws-ecs-service

**＊実装例＊**

```yaml
version: 2.1

orbs:
  aws-cli: circleci/aws-cli@1.3.1
  aws-ecs: circleci/aws-ecs@2.2.1
  
jobs:
  aws-ecs/deploy-update-service:
    name: ecs_update_service_by_rolling_update
    # タスク定義名を指定
    family: "${SERVICE}-ecs-task-definition"
    # ECSクラスター名を指定
    cluster-name: "${SERVICE}-cluster"
    # サービス名を指定
    service-name: "${SERVICE}-service"
    # コンテナ定義のコンテナ名とイメージタグを上書き．イメージはCircleCIのハッシュ値でタグ付けしているので必須．
    container-image-name-updates: "container=laravel,tag=${CIRCLE_SHA1},container=nginx,tag=${CIRCLE_SHA1}"
    # タスク定義に基づくタスク数の監視
    verify-revision-is-deployed: true
    # 監視の試行回数
    max-poll-attempts: 30
    # 試行の間隔
    poll-interval: 20
          
workflows:
  # ステージング環境にデプロイ
  develop:
    jobs:
      - ecs_update_service_by_rolling_update:
          name: ecs_update_service_by_rolling_update_stg
          filters:
            branches:
              only:
                - develop
                
  # 本番環境にデプロイ                
  main:
    jobs:
      - ecs_update_service_by_rolling_update:
          name: ecs_update_service_by_rolling_update_production
          filters:
            branches:
              only:
                - main               
          
```

#### ・deploy-update-service（ブルー/グリーンデプロイメント使用時）

ECSタスク定義を更新する．さらに，ブルー/グリーンデプロイメントがそのタスク定義を指定し，ECSサービスを更新する．ローリングアップデートと同様にして，``` verify-revision-is-deployed```オプションを用いることができる．

**＊実装例＊**

```yaml
version: 2.1

orbs:
  aws-cli: circleci/aws-cli@1.3.1
  aws-ecs: circleci/aws-ecs@2.2.1
  
jobs:
  aws-ecs/deploy-update-service:
    name: ecs_update_service_by_code_deploy
    # タスク定義名を指定
    family: "${SERVICE}-ecs-task-definition"
    # ECSクラスター名を指定
    cluster-name: "${SERVICE}-cluster"
    # サービス名を指定
    service-name: "${SERVICE}-service"
    # CodeDeployにおけるデプロイの作成を設定
    deployment-controller: CODE_DEPLOY
    codedeploy-application-name: $SERVICE
    codedeploy-deployment-group-name: "${SERVICE}-deployment-group"
    codedeploy-load-balanced-container-name: www-container
    codedeploy-load-balanced-container-port: 80
    # コンテナ名とイメージタグを指定．イメージはCircleCIのハッシュ値でタグ付けしているので必須．
    container-image-name-updates: "container=laravel,tag=${CIRCLE_SHA1},container=nginx,tag=${CIRCLE_SHA1}"
    # サービス更新後のタスク監視
    verify-revision-is-deployed: true
          
workflows:
  # ステージング環境にデプロイ
  develop:
    jobs:
      - ecs_update_service_by_code_deploy:
          name: ecs_update_service_by_code_deploy_stg
          filters:
            branches:
              only:
                - develop
                
  # 本番環境にデプロイ                
  main:
    jobs:
      - ecs_update_service_by_code_deploy:
          name: ecs_update_service_by_code_deploy_production
          filters:
            branches:
              only:
                - main       
```

#### ・run-task

現在起動中のECSタスクとは別に，新しいタスクを一時的に起動する．起動時に，```overrides```オプションを用いて，指定したタスク定義のコンテナ設定を上書きできる．正規表現で設定する必要があり，さらにJSONでは『```\```』を『```\\```』にエスケープしなければならない．コマンドが実行された後に，タスクは自動的にStopped状態になる．

上書きできるキーの参照リンク：https://docs.aws.amazon.com/cli/latest/reference/ecs/run-task.html

**＊実装例＊**

例えば，DBに対してマイグレーションを実行するためのECSタスクを起動する．```overrides```オプションでコンテナ定義のコマンドを上書きする．

```yaml
version: 2.1

orbs:
  aws-cli: circleci/aws-cli@1.3.1
  aws-ecs: circleci/aws-ecs@2.2.1

jobs:
  aws-ecs/run-task:
    name: ecs_run_task_for_migration
    cluster: "${SERVICE}-ecs-cluster"
    # LATESTとするとその時点の最新バージョンを自動で割り振られてしまう．
    platform-version: 1.4.0
    awsvpc: true
    launch-type: FARGATE
    subnet-ids: $AWS_SUBNET_IDS
    security-group-ids: $AWS_SECURITY_GROUPS
    # タスク定義名．最新リビジョン番号が自動補完される．
    task-definition: "${SERVICE}-ecs-task-definition"
    # タスク起動時にマイグレーションコマンドを実行するように，Laravelコンテナの　commandキーを上書き
    overrides: "{\\\"containerOverrides\\\":[{\\\"name\\\": \\\"laravel-container\\\",\\\"command\\\": [\\\"php\\\", \\\"artisan\\\", \\\"migrate\\\", \\\"--force\\\"]}]}"
          
workflows:
  # ステージング環境にデプロイ
  develop:
    jobs:
      - ecs_run_task_for_migration:
          name: ecs_run_task_for_migration_stg
          filters:
            branches:
              only:
                - develop
                
  # 本番環境にデプロイ                
  main:
    jobs:
      - ecs_run_task_for_migration:
          name: ecs_run_task_for_migration_production
          filters:
            branches:
              only:
                - main
```

<br>

## 05. aws-code-deploy

### jobs

#### ・deploy

S3にコードとappspecファイルをデプロイできる．また，CodeDeployを用いて，これをEC2インスタンスにデプロイできる．

**＊実装例＊**

```yaml
version: 2.1

orbs:
  aws-code-deploy: circleci/aws-code-deploy@1.0.1

jobs:
  aws-code-deploy/deploy:
    name: code_deploy
    application-name: $SERVICE}
    # appspecファイルを保存するバケット名
    bundle-bucket: "${SERVICE}-bucket"
    # appspecファイルのあるフォルダー
    bundle-source: ./infra/aws_codedeploy
    # appspecファイルをzipフォルダーで保存
    bundle-type: zip
    # zipフォルダー名
    bundle-key: foo-bundle
    deployment-config: CodeDeployDefault.ECSAllAtOnce
    deployment-group: "${SERVICE}-deployment-group"
    # ECSにアクセスできるCodeDeployサービスロール
    service-role-arn: $CODE_DEPLOY_ROLE_FOR_ECS
 
workflows:
  # ステージング環境にデプロイ
  develop:
    jobs:
      - code_deploy:
          name: code_deploy_stg
          filters:
            branches:
              only:
                - develop
                
  # 本番環境にデプロイ                
  main:
    jobs:
      - code_deploy:
          name: code_deploy_production
          filters:
            branches:
              only:
                - main
```

<br>

## 06. slack

### commands

#### ・notify

ジョブの終了時に，成功または失敗を基に，ステータスを通知する．ジョブの最後のステップとして設定しなければならない．

```yaml
version: 2.1

orbs:
  slack: circleci/slack@4.1

commands:
  # 他のジョブ内で用いることができるようにcommandとして定義
  notify_of_failure:
    steps:
      - slack/notify:
          event: fail
          template: basic_fail_1

jobs:
  deploy:
    steps:
    # ～ 中略 ～

workflows:
  # ステージング環境にデプロイ
  develop:
    jobs:
      - deploy:
          name: deploy_stg
          filters:
            branches:
              only:
                - develop
          # 失敗時に通知
          post-steps:
            - notify_of_failure:
            
  # 本番環境にデプロイ                
  main:
    jobs:
      - deploy:
          name: deploy_production
          filters:
            branches:
              only:
                - main
          # 失敗時に通知
          post-steps:
            - notify_of_failure:
```
