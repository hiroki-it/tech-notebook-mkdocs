---
title: 【IT技術の知見】テレメトリー間の紐付け＠Datadog
description: テレメトリー間の紐付け＠Datadog
---

# テレメトリー間の紐付け＠Datadog

## はじめに

本サイトにつきまして、以下をご認識のほど宜しくお願いいたします。

ℹ️ 参考：https://hiroki-it.github.io/tech-notebook-mkdocs/about.html

<br>

## 01. タグ

### タグの種類

ℹ️ 参考：

- https://docs.datadoghq.com/getting_started/tagging/
- https://www.datadoghq.com/ja/blog/tagging-best-practices/

| タグ名        | 説明                                                         |
| ------------- | ------------------------------------------------------------ |
| ```host```    | メトリクス、ログ、分散トレースの送信元のホスト名を示す。テレメトリーが作成元とは別の場所から受信した場合に役立つ。 |
| ```device```  |                                                              |
| ```source```  | ログの作成元のベンダー名を示す。                             |
| ```service``` | メトリクス、ログ、分散トレースの作成元のアプリケーション名を示す。 |
| ```env```     | メトリクス、ログ、分散トレースの作成元の実行環境名を示す。   |
| ```version``` | メトリクス、ログ、分散トレースの作成元のリリースバージョンを示す。 |

<br>

### 統合タグ付け

統合タグ（```service```、```env```、```version```）に同じ値を割り当てると、テレメトリー間を紐付けられる。

ℹ️ 参考：https://docs.datadoghq.com/getting_started/tagging/unified_service_tagging/?tab=kubernetes

<br>

### 各コンソール画面での使い方

ℹ️ 参考：https://docs.datadoghq.com/getting_started/tagging/using_tags/

<br>

## 01-02. 命名規則例

### 分散トレースのタグ

#### ▼ セットアップ

トレーシングパッケージを導入したコンテナの環境変数で値を設定する。

```yaml
{

    # ～ 中略 ～

    "environment": [
      {
        "name": "DD_ENV",
        "value": "prd"
      },
      {
        "name": "DD_SERVICE_NAME", # フレームワーク
        "value": "order"
      },
      {
        "name": "DD_SERVICE_MAPPING", # フレームワーク以外
        "value": "pdo:order-eloquent"
      },
      {
        "name": "DD_VERSION",
        "value": "1.0.0"
      }
    ],
    
    # ～ 中略 ～
}
```

#### ▼ service

分散トレースの作成元のアプリケーション名を表す。

| 規則                                                       | 例                                                           |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| ```<マイクロサービス名>-<フレームワーク名/パッケージ名>``` | ・```order-pdo```<br>・```order-phpredis```<br>・```order-eloquent```<br>・```order-doctrine``` |

ただし、フレームワークのインテグレーション名は接尾辞を付けずに、単に『```<マイクロサービス名>```』とする。なぜ接尾辞を付けないかというと、分散トレースと他のテレメトリーを紐づける時に、同じマイクロサービス名をタグに使用しなければならないためである。

〇：laravel -> order、order-service
✕：laravel -> order-laravel、order-service-laravel

フレームワーク以外のパッケージなどの分散トレースに関しては、紐づけられないことを許容している。APMのservice名とECSタスク/ログのserviceタグは、名前の付け方が異なることに注意する。service名はインテグレーション名が自動で割り当てられるが、トレーサーに環境変数を渡して変更もできる。フレームワークのインテグレーション名は `DD_SERVICE_NAME` から設定する一方で、それ以外のタグ名は `DD_SERVICE_MAPPING` から設定する。

ℹ️ 参考：https://docs.datadoghq.com/tracing/setup/php/#%E3%82%A4%E3%83%B3%E3%83%86%E3%82%B0%E3%83%AC%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E5%90%8D

#### ▼ env

分散トレースの作成元の環境名を表す。envタグは、datadogコンテナの環境変数を使用してタグ付けする。

| 規則           | 例                         |
| -------------- | -------------------------- |
| ```<環境名>``` | ・```prd```<br>・```stg``` |

#### ▼ version

リリース時のタグのバージョンを表す。ただし、リリースの度に変更することが面倒なため、```1.0.0```で固定する。

| 規則                             | 例          |
| -------------------------------- | ----------- |
| ```<リリースタグのバージョン>``` | ```1.0.0``` |

<br>

### ECSコンテナのタグ

#### ▼ セットアップ

Dockerラベルからタグ付けする。

#### ▼ env

コンテナの環境名を表す。

| 規則           | 例                         |
| -------------- | -------------------------- |
| ```<環境名>``` | ・```prd```<br>・```stg``` |

#### ▼ service

コンテナのアプリケーション名を表す。

| 規則               | 例                                         |
| ------------------ | ------------------------------------------ |
| ```<マイクロサービス名>``` | ・```order-service```<br>・```account-service``` |

#### ▼ version

リリース時のタグのバージョンを表す。ただし、リリースの度に変更することが面倒なため、```1.0.0```で固定する。

| 規則                             | 例          |
| -------------------------------- | ----------- |
| ```<リリースタグのバージョン>``` | ```1.0.0``` |

<br>

### ログのタグ

#### ▼ セットアップ

FluentBitの設定ファイルからタグ付けする。

#### ▼ env

ログの作成元の環境名を表す。

| 規則           | 例                         |
| -------------- | -------------------------- |
| ```<環境名>``` | ・```prd```<br>・```stg``` |

#### ▼ service

ログの作成元のアプリケーション名を表す。

| 規則               | 例                                                           |
| ------------------ | ------------------------------------------------------------ |
| ```<マイクロサービス名>``` | ・```order```<br>・```order-service```<br>・```account```<br>・```account-service``` |

#### ▼ source

ログの作成元のベンダー名を表す。

| 規則             | 例                                                     |
| ---------------- | ------------------------------------------------------ |
| ```<ソース名>``` | ・```laravel```<br>・```nginx```<br>・```apigateway``` |

<br>

### タグの候補

#### ▼ role

もし、```web```や```app```といった役割名をタグとして付与したい場合は、```role```タグを新たに作成すると良いかも。

<br>

## 02. 構造化ログと他テレメトリー間の紐付け

### 分散トレース全体との紐付け

スパンと構造化ログの統合タグ（```service```、```env```、```version```）に同じ値を割り当てると、分散トレース全体と構造化ログ間を紐付けられる。

ℹ️ 参考：https://docs.datadoghq.com/tracing/connect_logs_and_traces/

<br>

### スパンとの紐付け

スパンと構造化ログに、同じトレースIDとスパンIDを割り当てると、スパンと構造化ログ間を紐付けられる。これにより、その構造化ログが、いずれのマイクロサービスで、またどのタイミングで発生したものかを確認できる。

ℹ️ 参考：https://docs.datadoghq.com/tracing/visualization/trace/?tab=logs

![datadog_trace-viewer](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/datadog_trace-viewer.png)

<br>

## 03. メトリクスと他テレメトリー間の紐付け

### 仮想環境のメトリクスとの紐付け

スパンとコンテナのDockerLabelの統合タグ（```service```、```env```、```version```）に、同じ値を割り当てると、分散トレースと仮想環境のOSに関するメトリクスを紐付けられる。
