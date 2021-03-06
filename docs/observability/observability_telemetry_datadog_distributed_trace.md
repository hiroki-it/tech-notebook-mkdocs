---
title: 【IT技術の知見】分散トレース収集＠Datadog
description: 分散トレース収集＠Datadogの知見を記録しています。
---

# 分散トレース収集＠Datadog

## はじめに

本サイトにつきまして、以下をご認識のほど宜しくお願いいたします。

ℹ️ 参考：https://hiroki-it.github.io/tech-notebook-mkdocs/about.html

<br>

## 01. Traceエージェント（サーバーの場合）

### Traceエージェントとは

デーモンであるdatadogエージェントに含まれている。アプリケーションから分散トレースを収集し、Datadogに転送する。

ℹ️ 参考：https://www.netone.co.jp/knowledge-center/netone-blog/20210716-1/

![datadog-agent_on-server](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/datadog-agent_on-server.png)

<br>

### セットアップ

#### ▼ ```/etc/datadog-agent/datadog.yaml```ファイル

ℹ️ 参考：https://hiroki-it.github.io/tech-notebook-mkdocs/observability/observability_telemetry_datadog_agent_conf.html

<br>

## 02. Traceエージェント（AWS ECS Fargateの場合）

### Traceエージェントとは

サーバーの場合と同様にして、アプリケーションから分散トレースを受信し、Datadogに転送する。サーバーの場合とは異なり、自身が収集しにいくことはできない。仕組みとして、アプリケーションコンテナのトレースパッケージは分散トレースを作成し、datadogコンテナの『```http://localhost:8126```』にこれを送信する。datadogコンテナ内のdatadogエージェントはこれをHTTPSでDatadogに転送する。

ℹ️ 参考：

- https://docs.datadoghq.com/tracing/#datadog-%E3%81%B8%E3%83%88%E3%83%AC%E3%83%BC%E3%82%B9%E3%82%92%E9%80%81%E4%BF%A1
- https://inokara.hateblo.jp/entry/2017/10/01/164446

![datadog-tracer](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/datadog-tracer.png)

<br>

### セットアップ

#### ▼ パッケージ一覧

ℹ️ 参考：https://docs.datadoghq.com/developers/libraries/#apm-%E3%81%A8%E5%88%86%E6%95%A3%E5%9E%8B%E3%83%88%E3%83%AC%E3%83%BC%E3%82%B7%E3%83%B3%E3%82%B0%E3%82%AF%E3%83%A9%E3%82%A4%E3%82%A2%E3%83%B3%E3%83%88%E3%83%A9%E3%82%A4%E3%83%96%E3%83%A9%E3%83%AA

#### ▼ デバッグ

| 方法                      | 説明                                                         | 補足                                                         |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 起動ログの有効化          | 環境変数の```DD_TRACE_STARTUP_LOGS```を有効化することにより、起動ログを標準出力に出力できるようにする。起動ログから、トレーサーの設定値を確認できる。 | ℹ️ 参考：https://docs.datadoghq.com/tracing/troubleshooting/#troubleshooting-data-requested-by-datadog-support |
| デバッグログの有効化      | 各トレーサーが持つデバッグパラメーターを有効化することにより、デバッグログを標準出力に出力できるようにする。デバッグログから、実際にDatadogに送信されるスパンデータを確認できる。 | ℹ️ 参考：https://docs.datadoghq.com/tracing/troubleshooting/#troubleshooting-data-requested-by-datad |
| Agent Flareコマンドの実行 | datadogコンテナ内でAgent Flareコマンドを実行し、Datadogサポートにdatadogコンテナの構成情報をメール送信する。 | ℹ️ 参考：<br>・https://docs.datadoghq.com/tracing/troubleshooting/#troubleshooting-data-requested-by-datad<br>・https://docs.datadoghq.com/agent/troubleshooting/send_a_flare/?tab=agentv6v7 |

<br>

## 02-02. PHPトレーサー

### セットアップ

#### ▼ インストール（手動の場合）

使用しているミドルウェアごとに、インストール方法が異なる。サーバーを冗長化している場合、全てのサーバーに共通した設定のエージェントを組み込めるという点で、IaCツールを使用した方が良い。

```bash
# GitHubリポジトリからパッケージをダウンロードする。
$ curl -Lo https://github.com/DataDog/dd-trace-php/releases/download/0.63.0/datadog-php-tracer_0.63.0_amd64.deb

# パッケージをインストールをする。
$ dpkg -i datadog-php-tracer_0.69_amd64.deb

# 残骸ファイルを削除する。
$ rm datadog-php-tracer.deb
```

また、PHP-FPMに環境変数を渡せるように、```www```プールに関する設定ファイルを配置し、PHP-FPMを再起動する。

```ini
# /etc/php-fpm.d/dd-trace.confファイル
[www]
env[DD_SERVICE] = 'foo'
env[DD_SERVICE_MAPPING] = 'guzzle:foo-guzzle,pdo:foo-pdo'
env[DD_ENV] = 'prd'
env[DD_VERSION] = '<バージョンタグ>'
```

ℹ️ 参考：

- https://docs.datadoghq.com/tracing/setup/php/
- https://app.datadoghq.com/apm/docs?architecture=host-based&framework=php-fpm&language=php

#### ▼ インストール（Ansibleの場合）

使用しているミドルウェアごとに、インストール方法が異なる。

```yaml
- tasks:
    - name: Install dd-trace-php
      ansible.builtin.shell: |
        curl -Lo https://github.com/DataDog/dd-trace-php/releases/download/${DD_TRACE_VERSION}/datadog-php-tracer_${DD_TRACE_VERSION}_amd64.deb
        dpkg -i datadog-php-tracer_0.69_amd64.deb
        rm datadog-php-tracer.deb
      environment:
        DD_TRACE_VERSION: 0.63.0
    - name: Upload dd-trace.conf
      ansible.builtin.template: src=dd-trace.conf dest=/etc/php-fpm.d/dd-trace.conf
      notify: restart php-fpm
```

#### ▼ インストール（コンテナの場合）

アプリケーションコンテナのDockerfileにて、PHPトレーサーをインストールする。また、コンテナの環境変数として、```DD_SERVICE```、```DD_ENV```、```DD_VERSION```を渡す。

ℹ️ 参考：https://docs.datadoghq.com/tracing/setup_overview/setup/php/?tab=containers

```dockerfile
ENV DD_TRACE_VERSION=0.63.0

# GitHubリポジトリからパッケージをダウンロードする。
RUN curl -Lo https://github.com/DataDog/dd-trace-php/releases/download/${DD_TRACE_VERSION}/datadog-php-tracer_${DD_TRACE_VERSION}_amd64.deb \
  # パッケージをインストールする。
  && dpkg -i datadog-php-tracer.deb \
  # 残骸ファイルを削除する。
  && rm datadog-php-tracer.deb
```

#### ▼ インストールの動作確認

パッケージが正しく読み込まれているか否かは、```php --ri=ddtrace```コマンドまたは```phpinfo```メソッドの結果から確認できる。

```bash
# 成功の場合
$ php --ri=ddtrace

ddtrace


Datadog PHP tracer extension
For help, check out the documentation at https://docs.datadoghq.com/tracing/languages/php/
(c) Datadog 2020

# 〜 中略 〜

```

```bash
# 失敗の場合
$  php --ri=ddtrace
Extension 'ddtrace' not present.
```

#### ▼ パラメータの動作確認

パラメーターがトレーサーに渡されたか否かは、```DATADOG TRACER CONFIGURATION```の項目で確認できる。

ℹ️ 参考：https://docs.datadoghq.com/tracing/troubleshooting/tracer_startup_logs/

```bash
$ php --ri=ddtrace

Datadog tracing support => enabled
Version => 0.57.0
DATADOG TRACER CONFIGURATION => { ..... } # ここに設定のJSONが得られる

# 得られたJSONを整形している
{
    "date": "2021-00-00T09:00:00Z",
    "os_name": "Linux ***** 5.10.25-linuxkit #1 SMP Tue Mar 23 09:27:39 UTC 2021 x86_64",
    "os_version": "5.10.25-linuxkit",
    "version": "0.64.1",
    "lang": "php",
    "lang_version": "8.0.8",
    "env": null,
    "enabled": true,
    "service": null,
    "enabled_cli": false,
    "agent_url": "http://localhost:8126", # datadogコンテナのアドレスポート
    "debug": false,
    "analytics_enabled": false,
    "sample_rate": 1.000000,
    "sampling_rules": null,
    "tags": {},
    "service_mapping": {},
    "distributed_tracing_enabled": true,
    "priority_sampling_enabled": true,
    "dd_version": null,
    "architecture": "x86_64",
    "sapi": "cli",
    "datadog.trace.request_init_hook": "/opt/datadog-php/dd-trace-sources/bridge/dd_wrap_autoloader.php",
    "open_basedir_configured": false,
    "uri_fragment_regex": null,
    "uri_mapping_incoming": null,
    "uri_mapping_outgoing": null,
    "auto_flush_enabled": false,
    "generate_root_span": true,
    "http_client_split_by_domain": false,
    "measure_compile_time": true,
    "report_hostname_on_root_span": false,
    "traced_internal_functions": null,
    "auto_prepend_file_configured": false,
    "integrations_disabled": "default",
    "enabled_from_env": true,
    "opcache.file_cache": null,
    "agent_error": "Failed to connect to localhost port 8126: Connection refused", # エラーメッセージ
    "DDTRACE_REQUEST_INIT_HOOK": "'DDTRACE_REQUEST_INIT_HOOK=/opt/datadog-php/dd-trace-sources/bridge/dd_wrap_autoloader.php' is deprecated, use DD_TRACE_REQUEST_INIT_HOOK instead."
}
```

#### ▼ 受信ログの確認

datadogコンテナにトレースが送信されている場合は、受信できていることを表すログを確認できる。

```log
2022-01-01 12:00:00 UTC | TRACE | INFO | (pkg/trace/info/stats.go:111 in LogStats) | [lang:php lang_version:8.0.8 interpreter:fpm-fcgi tracer_version:0.64.1 endpoint_version:v0.4] -> traces received: 7, traces filtered: 0, traces amount: 25546 bytes, events extracted: 0, events sampled: 0
```



<br>

## 02-03. Node.jsトレーサー

### セットアップ

#### ▼ インストール

TypeScriptやモジュールバンドルを使用している場合、パッケージの読み出し処理が巻き上げられ、意図しない読み出しの順番になってしまうことがある。対策として、```dd-trace```パッケージの```init```メソッドの実行をを別ファイルに分割し、これをエントリーポイント（```nuxt.config.js```ファイル）の先頭で読み込むようにする。また、フレームワークよりも先に読み込むことになるため、```.env```ファイル参照機能を使えない。そこで、環境変数はインフラ側で設定する。

ℹ️ 参考：https://docs.datadoghq.com/tracing/setup_overview/setup/nodejs/?tab=%E3%82%B3%E3%83%B3%E3%83%86%E3%83%8A#typescript-%E3%81%A8%E3%83%90%E3%83%B3%E3%83%89%E3%83%A9%E3%83%BC


```javascript
// datadogTracer.tsファイル
import tracer from 'dd-trace'

tracer.init({
  // フレームワークの.envファイル参照機能を使用できない 
  env: DD_ENV,
  service: DD_SERVICE + '-ssr',
  version: DD_VERSION,
    
  // 検証時のオプション
  debug: true,
  startupLogs: true,
})

export default datadogTracer
```

```javascript
// nuxt.config.tsファイル
// 先頭で読み込む
import './datadogTracer'
import { Configuration } from '@nuxt/types'

// ～ 中略 ～
```

#### ▼ 起動ログの確認

トレーサーの起動ログは、```init```メソッドの```startupLogs```オプションを有効化すると確認できる。

```bash
DATADOG TRACER CONFIGURATION -
{
    "date": "2022-01-02T00:00:00.541Z",
    "os_name": "Darwin",
    "os_version": "20.6.0",
    "architecture": "arm64",
    "version": "2.0.1",
    "lang": "nodejs",
    "lang_version": "14.18.2",
    "env": "prd",
    "service": "foo",
    "agent_url": "http://127.0.0.1:8126",
    "agent_error": "Network error trying to reach the agent: socket hang up",
    "debug": false,
    "sample_rate": 1,
    "sampling_rules": [],
    "tags": {
        "service": "foo",
        "env": "prd",
        "version": "<バージョンタグ>",
        "runtime-id": "*****"
    },
    "dd_version": "<バージョンタグ>",
    "log_injection_enabled": false,
    "runtime_metrics_enabled": false,
    "profiling_enabled": false,
    "integrations_loaded": [
        "connect@3.7.0",
        "fs",
        "http",
        "https"
    ],
    "appsec_enabled": false
}

WARN  DATADOG TRACER DIAGNOSTIC - Agent Error: Network error trying to reach the agent: socket hang up 
```

<br>

### 環境変数

初期化時に環境変数を設定できる。APMのマイクロサービスのタグ名に反映される。

ℹ️ 参考：https://docs.datadoghq.com/tracing/setup_overview/setup/nodejs/?tab=%E3%82%B3%E3%83%B3%E3%83%86%E3%83%8A#%E3%82%B3%E3%83%B3%E3%83%95%E3%82%A3%E3%82%AE%E3%83%A5%E3%83%AC%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3

<br>

## 03. 分散トレースの作成

### 分散トレース

#### ▼ 分散トレースとは

ℹ️ 参考：https://hiroki-it.github.io/tech-notebook-mkdocs/observability/observability.html

#### ▼ Datadogにおける分散トレース

Datadogで、分散トレースはスパンを持つ配列データとして定義される。

ℹ️ 参考：https://docs.datadoghq.com/tracing/guide/send_traces_to_agent_by_api/

```yaml
[
    span1,
    span2,
    span3
]
```

また、複数の分散トレースを配列データとして定義できる。

```yaml
[
    trace1,
    trace2,
    trace3
]
```

<br>

### スパン

#### ▼ スパンとは

ℹ️ 参考：https://hiroki-it.github.io/tech-notebook-mkdocs/observability/observability.html

#### ▼ Datadogにおけるスパン

Datadogで、スパンはJSON型データとして定義される。アプリケーション内のトレーサーで、指定されたJSON型のスパンが作成され、スパンはdatadog-APIに送信される。

ℹ️ 参考：https://docs.datadoghq.com/tracing/guide/send_traces_to_agent_by_api/

**＊実装例＊**

```yaml
[
  [
    {
      "duration": 123,           # 処理の所要時間
      "error": 0,                # エラーの有無
      "meta": {
        "env": "prd"             # タグのり
      },
      "metrics": {
        "baz-sum": 123           # マイクロサービスのメトリクス
      },
      "name": "laravel.request", # スパン名
      "parent_id": 123,          # 親スパンID
      "resource": "/foo",        # アクセスされたリソース
      "service": "laravel",      # マイクロサービス名
      "span_id": 123456789,      # スパンID
      "start": 0,                # 処理開始時間
      "trace_id": 123456789,     # トレースID
      "type": "web"              # マイクロサービスのタイプ
    }
  ]
]

```

#### ▼ メタデータ

スパンの```meta```キーにメタデータのセットを割り当てられる。メタデータはタグとして機能する。

**＊実装例＊**

PHPトレーサーでlaravel内からタグを収集した例

```yaml
{
    "env": "prd",
    "http": {
        "host": "example.com",
        "method": "GET",
        "path_group": "/foo",
        "status_code": 200,
        "url": "https://example.com/foo/1"
    },
    "laravel": {
        "route": {
            "action": "App\Http\Controllers\Foo\FooController@get",
            "name": "foos.get"
        }
    },
    "php" : {
        "compilation": {
            "total_time_ms": 123.45
        }
    },
    "process_id": 100
}
```

<br>

### スパンのメトリクス

#### ▼ スパンのデータポイント化

スパンの持つデータをデータポイントとして集計すると、メトリクスのデータポイントを収集できる。

ℹ️ 参考：https://docs.datadoghq.com/tracing/generate_metrics/

#### ▼ メトリクス名の構成要素

メトリクス名は『```trace.<スパン名>.<メトリクス接尾辞名>```』の名前で構成される。

ℹ️ 参考：https://docs.datadoghq.com/tracing/guide/metrics_namespace/

#### ▼ メトリクスのスパン名

データポイントとなったスパン名が割り当てられる。

**＊例＊**

- ```trace.web.request.<メトリクス接尾辞名>```
- ```trace.db.query.<メトリクス接尾辞名>```
- ```trace.db.commit.<メトリクス接尾辞名>```

#### ▼ メトリクスのメトリクス接尾辞名

メトリクスの種類に応じた接尾辞名が割り当てられる。

ℹ️ 参考：https://docs.datadoghq.com/tracing/guide/metrics_namespace/#%E3%83%A1%E3%83%88%E3%83%AA%E3%82%AF%E3%82%B9%E3%82%B5%E3%83%95%E3%82%A3%E3%83%83%E3%82%AF%E3%82%B9

**＊例＊**

- ```trace.<スパン名>.hits.*****```（該当スパンのヒット数）
- ```trace.<スパン名>.duration```（該当スパンの処理時間）
- ```trace.<スパン名>.duration.by.*****```（該当スパンの処理時間の割合）
- ```trace.<スパン名>.errors.*****```（該当スパンにおけるエラー数）

<br>

### エラートラッキング

#### ▼ エラートラッキングの仕組み

エントリす

ℹ️ 参考：https://docs.datadoghq.com/tracing/error_tracking/#how-datadog-error-tracking-works

<br>

## 04. マイクロサービスの識別

### マイクロサービスタイプ

#### ▼ マイクロサービスタイプとは

トレーサによって、マイクロサービスは『Web』『DB』『Cache』『Cache』の4つに分類される。各マイクロサービスの```span.type```属性に割り当てられるタイプ名から自動的に割り振られる。タイプ名の種類については、以下のリンクを参考にせよ。

ℹ️ 参考：

- https://github.com/DataDog/dd-trace-php/blob/master/src/api/Type.php
- https://docs.datadoghq.com/tracing/visualization/services_list/#%E3%82%B5%E3%83%BC%E3%83%93%E3%82%B9%E3%82%BF%E3%82%A4%E3%83%97

<br>

### マイクロサービスのタグ

#### ▼ マイクロサービスのタグとは

トレーサによって、マイクロサービスにタグを追加できる。PHPトレーサの各インテグレーションのコードについては以下のリンクを参考にせよ。コードから、PHPトレーサーがアプリケーションからどのように情報を抜き出し、分散トレースのタグの値を決定しているかがわかる。

ℹ️ 参考：

- https://github.com/DataDog/dd-trace-php/tree/master/src/Integrations/Integrations
- https://github.com/DataDog/dd-trace-php/blob/master/src/api/Tag.php
