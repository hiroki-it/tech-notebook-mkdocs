# FluentBit

## 01. ログパイプライン

### ログパイプラインとは

FluentBitにて、ログを処理する一連のセクションのこと。

![fluent-bit-log-pipeline](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/fluent-bit_log-pipeline.png)

<br>

## セクションの設定

#### ・confファイル

confファイルでセクションを設定できる。

参考：https://docs.fluentbit.io/manual/administration/configuring-fluent-bit/configuration-file

#### ・コマンド

コマンドでセクションを実行できる。

```bash
$ /fluent-bit/bin/fluent-bit --help

Available Options
  -b  --storage_path=PATH specify a storage buffering path
  -c  --config=FILE       specify an optional configuration file
  -d, --daemon            run Fluent Bit in background mode
  -D, --dry-run           dry run
  -f, --flush=SECONDS     flush timeout in seconds (default: 5)
  -F  --filter=FILTER     set a filter
  -i, --input=INPUT       set an input
  -m, --match=MATCH       set plugin match, same as '-p match=abc'
  -o, --output=OUTPUT     set an output
  -p, --prop="A=B"        set plugin configuration property
  -R, --parser=FILE       specify a parser configuration file
  -e, --plugin=FILE       load an external plugin (shared lib)
  -l, --log_file=FILE     write log info to a file
  -t, --tag=TAG           set plugin tag, same as '-p tag=abc'
  -T, --sp-task=SQL       define a stream processor task
  -v, --verbose           increase logging verbosity (default: info)
  -w, --workdir           set the working directory
  -H, --http              enable monitoring HTTP server
  -P, --port              set HTTP server TCP port (default: 2020)
  -s, --coro_stack_size   set coroutines stack size in bytes (default: 24576)
  -q, --quiet             quiet mode
  -S, --sosreport         support report for Enterprise customers
  -V, --version           show version number
  -h, --help              print this help
```

#### ・バリデーション

設定ファイルのバリデーションは、開発環境にて、以下サイトや再起動を伴う```--config```オプションから行う。これら以外に再起動を伴わない```--dry-run```オプションがあるが、このオプションは経験則で精度が低いため、参考程度にする。

参考：https://cloud.calyptia.com/visualizer

```bash
$ /fluent-bit/bin/fluent-bit --config=/fluent-bit/etc/fluent-bit_custom.conf
```

<br>

## 02. セクション

### SERVICE

#### ・SERVICEセクションとは

パイプライン全体の設定やファイルの読み込みを定義する。各設定の頭文字は大文字とする。

**＊実装例＊**

フラッシュについては、以下のリンクを参考にせよ。

参考：https://stackoverflow.com/questions/47735850/what-exactly-is-flushing

```bash
[SERVICE]
    # バッファに蓄えられた全てのログを宛先に出力する間隔
    Flush 1
    # 猶予時間
    Grace 30
    # FluentBit自体のログレベル
    Log_Level info
    # 読み込まれるParsers Multilineファイルの名前
    Parsers_File parsers_multiline.conf
    # 読み込まれるStream Processorファイルの名前
    Streams_File stream_processor.conf
```

#### ・実行ログの確認

Log_Level値でFluentBitのログレベルを制御できる。```debug```を割り当てると、FluentBitのログがより詳細になり、各セクションの設定値を確認できるようになる。

**＊実行ログ例＊**

```bash
Fluent Bit v1.8.6
* Copyright (C) 2019-2021 The Fluent Bit Authors
* Copyright (C) 2015-2018 Treasure Data
* Fluent Bit is a CNCF sub-project under the umbrella of Fluentd
* https://fluentbit.io

[2021/01/01 12:00:00] [ info] Configuration:
[2021/01/01 12:00:00] [ info]  flush time     | 1.000000 seconds
[2021/01/01 12:00:00] [ info]  grace          | 30 seconds
[2021/01/01 12:00:00] [ info]  daemon         | 0
[2021/01/01 12:00:00] [ info] ___________
[2021/01/01 12:00:00] [ info]  inputs:
[2021/01/01 12:00:00] [ info]      forward
[2021/01/01 12:00:00] [ info] ___________
[2021/01/01 12:00:00] [ info]  filters:
[2021/01/01 12:00:00] [ info]      stdout.0
[2021/01/01 12:00:00] [ info] ___________
[2021/01/01 12:00:00] [ info]  outputs:
[2021/01/01 12:00:00] [ info]      null.0
[2021/01/01 12:00:00] [ info] ___________
[2021/01/01 12:00:00] [ info]  collectors:
[2021/01/01 12:00:00] [ info] [engine] started (pid=1)
[2021/01/01 12:00:00] [debug] [engine] coroutine stack size: 24576 bytes (24.0K)
[2021/01/01 12:00:00] [debug] [storage] [cio stream] new stream registered: forward.0
[2021/01/01 12:00:00] [ info] [storage] version=1.1.1, initializing...
[2021/01/01 12:00:00] [ info] [storage] in-memory
[2021/01/01 12:00:00] [ info] [storage] normal synchronization mode, checksum disabled, max_chunks_up=128
[2021/01/01 12:00:00] [ info] [cmetrics] version=0.2.1
[2021/01/01 12:00:00] [debug] [in_fw] Listen='0.0.0.0' TCP_Port=24224
[2021/01/01 12:00:00] [ info] [input:forward:forward.0] listening on 0.0.0.0:24224
[2021/01/01 12:00:00] [debug] [null:null.0] created event channels: read=21 write=22
[2021/01/01 12:00:00] [debug] [router] match rule forward.0:null.0
[2021/01/01 12:00:00] [ info] [sp] stream processor started
```

<br>

### INPUT

#### ・INPUTセクションとは

ログのパイプラインへの入力方法を定義する。

参考：https://docs.fluentbit.io/manual/concepts/data-pipeline/input

プラグインを用いて、ログの入力方法を設定する。

参考：https://docs.fluentbit.io/manual/pipeline/inputs

コマンドの```-i```オプションでINPUT名を指定し、実行することもできる。

```bash
Inputs
  cpu                     CPU Usage
  mem                     Memory Usage
  thermal                 Thermal
  kmsg                    Kernel Log Buffer
  proc                    Check Process health
  disk                    Diskstats
  systemd                 Systemd (Journal) reader
  netif                   Network Interface Usage
  docker                  Docker containers metrics
  docker_events           Docker events
  node_exporter_metrics   Node Exporter Metrics (Prometheus Compatible)
  fluentbit_metrics       Fluent Bit internal metrics
  tail                    Tail files
  dummy                   Generate dummy data
  head                    Head Input
  health                  Check TCP server health
  http                    HTTP
  collectd                collectd input plugin
  statsd                  StatsD input plugin
  serial                  Serial input
  stdin                   Standard Input
  syslog                  Syslog
  exec                    Exec Input
  tcp                     TCP
  mqtt                    MQTT, listen for Publish messages
  forward                 Fluentd in-forward
  random                  Random
```

#### ・dummyプラグイン

ダミーの構造化ログをパイプラインに入力する。非構造化ログは入力データとして使用できない。ローカル環境でパイプラインの動作を確認するために役立つ。

参考：

- https://docs.fluentbit.io/manual/pipeline/inputs/dummy
- https://docs.fluentbit.io/manual/local-testing/logging-pipeline

```json
{
  "message": "dummy"
}
```

**＊実装例＊**

```bash
[INPUT]
    Name   dummy
    # ダミーJSONデータ
    Dummy  {"message":"dummy"}
```

**＊例＊**

```bash
$ /fluent-bit/bin/fluent-bit -i dummy -o stdout
```

#### ・forwardプラグイン

受信したログを指定されたポートでリッスンし、パイプラインに入力する。

参考：https://docs.fluentbit.io/manual/pipeline/inputs/forward

**＊実装例＊**

```bash
[INPUT]
    # プラグイン名
    Name        forward
    Listen      0.0.0.0
    # プロセスのリッスンポート
```

**＊実行ログ例＊**

```bash
Fluent Bit v1.8.6
* Copyright (C) 2019-2021 The Fluent Bit Authors
* Copyright (C) 2015-2018 Treasure Data
* Fluent Bit is a CNCF sub-project under the umbrella of Fluentd
* https://fluentbit.io

[2021/01/01 12:00:00] [ info] [engine] started (pid=1)
[2021/01/01 12:00:00] [ info] [storage] version=1.1.1, initializing...
[2021/01/01 12:00:00] [ info] [storage] in-memory
[2021/01/01 12:00:00] [ info] [storage] normal synchronization mode, checksum disabled, max_chunks_up=128
[2021/01/01 12:00:00] [ info] [cmetrics] version=0.2.1
[2021/01/01 12:00:00] [ info] [input:forward:forward.0] listening on 0.0.0.0:24224
[2021/01/01 12:00:00] [ info] [sp] stream processor started
```

**＊例＊**

```bash
$ /fluent-bit/bin/fluent-bit \
  -i forward \
  -o stdout
```

#### ・tailプラグイン

指定したパスに継続的に出力されるログファイルを順次結合し、パイプラインに入力する。あらかじめ、FluentBitコンテナ内にログファイルを配置する必要があり、```Path```でこれを指定する。```v1.8```を境にオプションが変わっていることに注意する。

参考：https://docs.fluentbit.io/manual/pipeline/inputs/tail

**＊実装例＊**

```bash
[INPUT]
    # プラグイン名
    Name              tail
    # FluentBitコンテナ内のログファイルの場所。ワイルドカードを使用できる。
    Path              /var/www/foo/storage/logs/*.log
    # 用いるパーサー名
    multiline.parser  laravel-multiline-parser
```

```yaml
log_router:
  container_name: fluentbit
  build:
    dockerfile: ./docker/fluentbit/Dockerfile
    context: .
  volumes:
    # アプリケーションのログファイルのボリュームマウント
    - ./storage/logs:/var/www/foo/storage/logs
```

**＊例＊**

参考：https://docs.fluentbit.io/manual/pipeline/inputs/tail#command-line

```bash
$ fluent-bit \
  -i tail \
  -p path=/var/www/foo/storage/logs/*.log \
  -o stdout
```

**＊実行ログ例＊**

```bash
* Copyright (C) 2019-2021 The Fluent Bit Authors
* Copyright (C) 2015-2018 Treasure Data
* Fluent Bit is a CNCF sub-project under the umbrella of Fluentd
* https://fluentbit.io

[2021/01/01 12:00:00] [ info] [engine] started (pid=1)
[2021/01/01 12:00:00] [ info] [storage] version=1.1.1, initializing...
[2021/01/01 12:00:00] [ info] [storage] in-memory
[2021/01/01 12:00:00] [ info] [storage] normal synchronization mode, checksum disabled, max_chunks_up=128
[2021/01/01 12:00:00] [ info] [cmetrics] version=0.2.1
[2021/01/01 12:00:00] [ info] [sp] stream processor started
[2021/01/01 12:00:00] [ info] [input:tail:tail.0] inotify_fs_add(): inode=31621169 watch_fd=1 name=/var/www/foo/storage/logs/laravel.log
[0] tail.0: [1634640932.010306200, {"log"=>"[2021-01-01 12:00:00] local.INFO: メッセージ"}]
[1] tail.0: [1634640932.013139300, {"log"=>"[2021-01-01 12:00:00] local.INFO: メッセージ"}]
[2] tail.0: [1634640932.013147300, {"log"=>"[2021-01-01 12:00:00] local.INFO: メッセージ"}]
```

<br>

### PARSE

#### ・PARSEセクションとは

#### ・MULTILINE_PARSERセクション

参考：https://docs.fluentbit.io/manual/administration/configuring-fluent-bit/multiline-parsing

**＊実装例＊**

Laravelのスタックトレースを結合する。

```bash
[MULTILINE_PARSER]
    # パーサー名
    name          laravel
    # パーサータイプ
    type          regex
    flush_timeout 1000
    # パーサールール。スタックトレースの文頭をstart_state、また以降に結合する文字列をcontで指定する。
    # [%Y-%m-%d %H:%M:%S] をスタックトレースの開始地点とする。
    rule          "start_state" "/\[[12]\d{3}-(0[1-9]|1[0-2])-(0[1-9]|[12]\d|3[01])\s+([01]?\d|2[0-3]):([0-5]\d):([0-5]\d)\].*/" "cont"
    # [stacktrace]、[previous exception]、#、行間、"} 、で始まる文字の場合に結合する。
    rule          "cont" "/(\[(stacktrace|previous exception)\]|#|\n\n|"\}).*/" "cont"
```

<br>

### FILTER

#### ・FILTERセクションとは

特定の文字列を持つログのみをBUFFERセクションに移行する。

#### ・multilineプラグイン

参考：https://docs.fluentbit.io/manual/pipeline/filters/multiline-stacktrace

```bash
[FILTER]
    # プラグイン名
    name                  multiline
    match                 *
    multiline.key_content log
    # 用いるパーサー名
    multiline.parser      laravel
```

コマンドの```-f```オプションでINPUT名を指定し、実行することもできる。

```bash
Filters
  alter_size              Alter incoming chunk size
  aws                     Add AWS Metadata
  checklist               Check records and flag them
  record_modifier         modify record
  throttle                Throttle messages using sliding window algorithm
  kubernetes              Filter to append Kubernetes metadata
  modify                  modify records by applying rules
  multiline               Concatenate multiline messages
  nest                    nest events by specified field values
  parser                  Parse events
  expect                  Validate expected keys and values
  grep                    grep events by specified field values
  rewrite_tag             Rewrite records tags
  lua                     Lua Scripting Filter
  stdout                  Filter events to STDOUT
  geoip2                  add geoip information to records
```

#### ・stdoutプラグイン

参考：https://docs.fluentbit.io/manual/pipeline/filters/standard-output

```bash
[FILTER]
    # プラグイン名
    Name  stdout
    Match *
```

**＊例＊**

```bash
$ /fluent-bit/bin/fluent-bit \
  -i <インプット名> \
  -F stdout \
  -m '*' \
  -o null
```

**＊実行ログ例＊**

```bash
Fluent Bit v1.8.6
* Copyright (C) 2019-2021 The Fluent Bit Authors
* Copyright (C) 2015-2018 Treasure Data
* Fluent Bit is a CNCF sub-project under the umbrella of Fluentd
* https://fluentbit.io

[2021/01/01 06:18:52] [ info] [engine] started (pid=40)
[2021/01/01 06:18:52] [ info] [storage] version=1.1.1, initializing...
[2021/01/01 06:18:52] [ info] [storage] in-memory
[2021/01/01 06:18:52] [ info] [storage] normal synchronization mode, checksum disabled, max_chunks_up=128
[2021/01/01 06:18:52] [ info] [cmetrics] version=0.2.1
[2021/01/01 06:18:52] [ info] [sp] stream processor started
[0] cpu.0: [1634710733.114477665, {"cpu_p"=>0.166667, "user_p"=>0.000000, "system_p"=>0.166667, "cpu0.p_cpu"=>0.000000, "cpu0.p_user"=>0.000000, "cpu0.p_system"=>0.000000, "cpu1.p_cpu"=>1.000000, "cpu1.p_user"=>0.000000, "cpu1.p_system"=>1.000000, "cpu2.p_cpu"=>0.000000, "cpu2.p_user"=>0.000000, "cpu2.p_system"=>0.000000, "cpu3.p_cpu"=>0.000000, "cpu3.p_user"=>0.000000, "cpu3.p_system"=>0.000000, "cpu4.p_cpu"=>0.000000, "cpu4.p_user"=>0.000000, "cpu4.p_system"=>0.000000, "cpu5.p_cpu"=>0.000000, "cpu5.p_user"=>0.000000, "cpu5.p_system"=>0.000000}]
[0] cpu.0: [1634710734.115201385, {"cpu_p"=>0.333333, "user_p"=>0.166667, "system_p"=>0.166667, "cpu0.p_cpu"=>0.000000, "cpu0.p_user"=>0.000000, "cpu0.p_system"=>0.000000, "cpu1.p_cpu"=>0.000000, "cpu1.p_user"=>0.000000, "cpu1.p_system"=>0.000000, "cpu2.p_cpu"=>0.000000, "cpu2.p_user"=>0.000000, "cpu2.p_system"=>0.000000, "cpu3.p_cpu"=>0.000000, "cpu3.p_user"=>0.000000, "cpu3.p_system"=>0.000000, "cpu4.p_cpu"=>0.000000, "cpu4.p_user"=>0.000000, "cpu4.p_system"=>0.000000, "cpu5.p_cpu"=>0.000000, "cpu5.p_user"=>0.000000, "cpu5.p_system"=>0.000000}]
[0] cpu.0: [1634710735.114646610, {"cpu_p"=>1.500000, "user_p"=>0.666667, "system_p"=>0.833333, "cpu0.p_cpu"=>0.000000, "cpu0.p_user"=>0.000000, "cpu0.p_system"=>0.000000, "cpu1.p_cpu"=>3.000000, "cpu1.p_user"=>2.000000, "cpu1.p_system"=>1.000000, "cpu2.p_cpu"=>2.000000, "cpu2.p_user"=>1.000000, "cpu2.p_system"=>1.000000, "cpu3.p_cpu"=>1.000000, "cpu3.p_user"=>0.000000, "cpu3.p_system"=>1.000000, "cpu4.p_cpu"=>1.000000, "cpu4.p_user"=>0.000000, "cpu4.p_system"=>1.000000, "cpu5.p_cpu"=>2.000000, "cpu5.p_user"=>1.000000, "cpu5.p_system"=>1.000000}]

```

<br>

### BUFFERセクション

#### ・BUFFERセクションとは

![buffering_chunk](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/buffering_chunk.png)

Fluentdから概念図を拝借した。バッファーとして機能するメモリ/ファイルにて、チャンク化されたログテキストは一旦ステージに蓄えられる。ステージに一定量のチャンクが蓄えられると、チャンクはキューに格納される。キューは、ログテキストを指定された形式でターゲットに順番にルーティングする。プロセスが再起動されると、メモリ/ファイルに蓄えられたログテキストは破棄されてしまう。ちなみに、AWS Kinesis Data Firehoseも似たようなバッファリングとルーティングの仕組みを持っている。

参考：

- https://docs.fluentbit.io/manual/administration/buffering-and-storage
- https://atmarkit.itmedia.co.jp/ait/articles/1402/06/news007.html
- https://www.alpha.co.jp/blog/202103_01

#### ・メモリ上でバッファリング

参考：https://docs.fluentbit.io/manual/administration/buffering-and-storage#input-section-configuration

**＊実装例＊**

```bash
[SERVICE]
    flush         1
    log_Level     info

[INPUT]
    name          cpu
    # メモリ上でバッファリングが実行される（デフォルト値）。
    storage.type  memory
```

#### ・ファイル上でバッファリング

参考：https://docs.fluentbit.io/manual/administration/buffering-and-storage#input-section-configuration

**＊実装例＊**

```bash
[SERVICE]
    flush         1
    log_Level     info
    # ファイルの場所
    storage.path  /var/log/fluentbit/

[INPUT]
    name          cpu
    # ファイル上でバッファリングが実行される。
    storage.type  filesystem
```

指定した場所に```cpu.0```ディレクトリが生成され、そこにあるflbファイル上でバッファリングが実行される。

```bash
$ ls -ls /var/log/fluentbit/cpu.0

-rw------- 1 root root 4096 Oct 20 15:51 1-1634745095.575805200.flb
```

<br>

### STREAM_TASKセクション

#### ・STREAM_TASKセクションとは

![fluent-bit_stream-task](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/fluent-bit_stream-task.png)

チャンク化されたログにタグ付けを行う。タグ付けされたログは、パイプラインのINPUTセクションに再度取り込まれ、処理し直される。

参考：https://docs.fluentbit.io/manual/stream-processing/overview#stream-processor

STREAM_TASKセッションは、ログSQLで定義される。

参考：https://docs.fluentbit.io/manual/stream-processing/getting-started/fluent-bit-sql

```bash
[STREAM_TASK]
    Name foo
    Exec CREATE STREAM foo AS SELECT * FROM TAG:'foo';
```

<br>

### OUTPUT

#### ・OUTPUTセクションとは

チャンクとして蓄えられたログの出力先を定義する。設定可能な出力先の種類については、以下を参考にせよ。

参考：https://docs.fluentbit.io/manual/pipeline/outputs

コマンドの```-o```オプションでINPUT名を指定し、実行することもできる。

```bash
Outputs
  azure                   Send events to Azure HTTP Event Collector
  azure_blob              Azure Blob Storage
  bigquery                Send events to BigQuery via streaming insert
  counter                 Records counter
  datadog                 Send events to DataDog HTTP Event Collector
  es                      Elasticsearch
  exit                    Exit after a number of flushes (test purposes)
  file                    Generate log file
  forward                 Forward (Fluentd protocol)
  http                    HTTP Output
  influxdb                InfluxDB Time Series
  logdna                  LogDNA
  loki                    Loki
  kafka                   Kafka
  kafka-rest              Kafka REST Proxy
  nats                    NATS Server
  nrlogs                  New Relic
  null                    Throws away events
  plot                    Generate data file for GNU Plot
  slack                   Send events to a Slack channel
  splunk                  Send events to Splunk HTTP Event Collector
  stackdriver             Send events to Google Stackdriver Logging
  stdout                  Prints events to STDOUT
  syslog                  Syslog
  tcp                     TCP Output
  td                      Treasure Data
  flowcounter             FlowCounter
  gelf                    GELF Output
  websocket               Websocket
  cloudwatch_logs         Send logs to Amazon CloudWatch
  kinesis_firehose        Send logs to Amazon Kinesis Firehose
  kinesis_streams         Send logs to Amazon Kinesis Streams
  prometheus_exporter     Prometheus Exporter
  prometheus_remote_write Prometheus remote write
  s3                      Send to S3
```

#### ・AWSの任意のリソースに出力

AWSから提供される他の全てのFluentBitイメージを束ねたベースイメージを用いる。

参考：https://github.com/aws/amazon-cloudwatch-logs-for-fluent-bit

#### ・Cloudwatchログへの出力

cloudwatch_logsプラグインがあらかじめインストールされているベースイメージを用いる。

参考：https://github.com/aws/amazon-cloudwatch-logs-for-fluent-bit

設定ファイルに予約されたAWS変数については、以下のリンクを参考にせよ。

参考：https://github.com/aws/amazon-cloudwatch-logs-for-fluent-bit#templating-log-group-and-stream-names

```bash
#########################
# CloudWatchログへのルーティング
#########################
[OUTPUT]
    # プラグイン名
    Name              cloudwatch_logs
    # ルーティング対象とするログのタグ
    Match             laravel
    # アウトプットJSONのうち、宛先にルーティングするキー名
    log_key           log
    region            ap-northeast-1
    # 予約変数あり。
    log_group_name    /prd-foo-ecs-container/laravel/log
    # ログストリーム名。予約変数あり。タスクIDなど出力できる。
    log_stream_name   container/laravel/$(ecs_task_id)
    
[OUTPUT]
    Name              cloudwatch_logs
    Match             nginx
    log_key           log
    region            ap-northeast-1
    log_group_name    /prd-foo-ecs-container/nginx/log
    log_stream_name   container/nginx/$(ecs_task_id)
```

CloudWatchログに送信されるデータはJSON型である。```log```キーにログが割り当てられている。特定のキーの値のみをCloudWatchログに送信する場合、log_keyオプションでキー名を設定する。例えば、```log```キーのみを送信する場合、『```log```』と設定する。

参考：https://blog.msysh.me/posts/2020/07/split_logs_into_multiple_target_with_firelens_and_rewrite_tag.html

```bash
{
    "container_id": "*****",
    "container_name": "prd-foo-ecs-container",
    "ecs_cluster": "prd-foo-ecs-cluster",
    "ecs_task_arn": "arn:aws:ecs:ap-northeast-1:****:task/cluster-name/*****",
    "ecs_task_definition": "prd-foo-ecs-task-definition:1",
    "log": "<ログ>",
    "source": "stdout",
    "ver": "1.5"
}
```

#### ・Datadogへの出力

全てのベースイメージにデフォルトでdatadogプラグインがインストールされているため、datadogプラグインのインストールは不要である。

参考：https://github.com/DataDog/fluent-plugin-datadog

```bash
#########################
# Datadogへのルーティング
#########################
[OUTPUT]
    # プラグイン名
    Name              datadog
    # ルーティング対象とするログのタグ
    Match             laravel
    # ルーティング先ホスト
    Host              http-intake.logs.datadoghq.com
    TLS               on
    compress          gzip
    # DatadogのAPIキー。
    apikey            *****
    # DatadogログエクスプローラーにおけるService名
    dd_service        prd-foo
    # DatadogログエクスプローラーにおけるSource名
    dd_source         prd-foo
    dd_message_key    log
    # 追加タグ
    dd_tags           env:prd-foo
    
[OUTPUT]
    Name              datadog
    Match             nginx
    Host              http-intake.logs.datadoghq.com
    TLS               on
    compress          gzip
    apikey            *****
    dd_service        prd-foo
    dd_source         prd-foo
    dd_message_key    log
    dd_tags           env:prd-foo
```

代わりに、同じ設定をFireLensの```logConfiguration```キーとしても適用することもできる。

参考：https://github.com/aws-samples/amazon-ecs-firelens-examples/blob/mainline/examples/fluent-bit/datadog/README.md

```bash
"logConfiguration": {
	"logDriver":"awsfirelens",
	"options": {
	   "Name": "datadog",
	   "Host": "http-intake.logs.datadoghq.com",
	   "TLS": "on",
	   "apikey": "<DATADOG_API_KEY>",
	   "dd_service": "prd-foo",
	   "dd_source": "prd-foo",
	   "dd_tags": "env:prd-foo",
	   "provider": "ecs"
   }
},
```

#### ・Kinesis Firehoseへの出力

kinesis_firehoseプラグインがあらかじめインストールされているベースイメージを用いる。

参考：https://github.com/aws/amazon-kinesis-firehose-for-fluent-bit

#### ・Kinesis Streamsへの出力

kinesis_streamsプラグインがあらかじめインストールされているベースイメージを用いる。

参考：https://github.com/aws/amazon-kinesis-streams-for-fluent-bit

#### ・NewRelicへの出力

newRelicプラグインがあらかじめインストールされているベースイメージを用いる。

参考：https://github.com/newrelic/newrelic-fluent-bit-output

#### ・標準出力への出力

標準出力に出力する、FluentBitの実行ログに混じって、対象のログが出力されることになる。

```bash
 [OUTPUT]
    Name   stdout
    match  *
```

#### ・破棄

出力を破棄する。

参考：https://docs.fluentbit.io/manual/pipeline/outputs/null

**＊実装例＊**

```bash
[OUTPUT]
    Name   null
    match  *
```

**＊例＊**

```bash
$ /fluent-bit/bin/fluent-bit \
  -i <インプット名> \
  -F stdout \
  -m '*' \
  -o null
```

<br>

## 03. Fargateコンテナからのログ収集

### FireLensコンテナ

#### ・FireLensコンテナとは

AWSが提供するFluentBit/Fluentdイメージによって構築されるコンテナであり、Fargateコンテナのサイドカーコンテナとして配置される。Fargateコンテナからログが送信されると、コンテナ内で稼働するFluentBit/Fluentdがこれを収集し、これを他のサービスにルーティングする。構築のための実装例については、以下のリンクを参考にせよ。

参考：

- https://github.com/aws-samples/amazon-ecs-firelens-examples
- https://aws.amazon.com/jp/blogs/news/announcing-firelens-a-new-way-to-manage-container-logs/

#### ・ログのルーティング先

FluentBit/Fluentdが対応する他のサービスにログをルーティングできる。

参考：https://docs.fluentbit.io/manual/pipeline/outputs

<br>

### サイドカーコンテナパターン

#### ・サイドカーコンテナパターンとは

サイドカーコンテナパターンを含むコンテナデザインパターンについては、以下のリンクを参考にせよ。

参考：https://hiroki-it.github.io/tech-notebook-mkdocs/md/virtualization/virtualization_container_orchestration.html

#### ・ログの収集/ルーティングの仕組み

以下の順番でログの収集/ルーティングを実行する。

参考：https://aws.amazon.com/jp/blogs/news/under-the-hood-firelens-for-amazon-ecs-tasks/

1. awsfirelensドライバーはFluentdログドライバーをラッピングしたものであり、ログをFireLensコンテナに送信する。Fluentdログドライバーについては、以下を参考にせよ。

   参考：https://docs.docker.com/config/containers/logging/fluentd/

2. FireLensコンテナは、これを受信する。

3. コンテナ内で稼働するFluentBitのログパイプラインのINPUTに渡され、FluentBitはログを処理する。

4. OUTPUTセクションに渡され、FluentBitは指定した外部サービスにログをルーティングする。

![fluent-bit_aws-firelens](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/fluent-bit_aws-firelens.png)

#### ・ログルーティングプロセス

FireLensコンテナでは、FluentBitまたはFlunetdがログルーティングプロセスとして稼働する。FireLensコンテナを使用せずに、独自のコンテナを構築して稼働させることも可能であるが、FireLensコンテナを用いれば、主要なセットアップがされているため、より簡単な設定でFluentBitまたはFlunetdを使用できる。FluentBitの方がより低負荷で稼働するため、FluentBitが推奨されている。

参考：

- https://aws.amazon.com/jp/blogs/news/under-the-hood-firelens-for-amazon-ecs-tasks/
- https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/userguide/using_firelens.html

<br>

### ベースイメージ

#### ・FluentBitイメージ

FireLensコンテナのベースイメージとなるFluentBitイメージがAWSから提供されている。AWSリソースにログをルーティングするためのプラグインがすでに含まれている。なお、DatadogプラグインはFluentBit自体にインストール済みである。パブリックECRリポジトリからプルしたイメージをそのまま用いる場合と、プライベートECRリポジトリで再管理してから用いる場合がある。

参考：https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/firelens-using-fluentbit.html

```bash
[root@<コンテナID>:/fluent-bit]$ ls -la

-rw-r--r-- 1 root root 26624256 Sep  1 18:04 cloudwatch.so # 旧cloudwatch_logsプラグイン
-rw-r--r-- 1 root root 26032656 Sep  1 18:04 firehose.so   # kinesis_firehoseプラグイン 
-rw-r--r-- 1 root root 30016544 Sep  1 18:03 kinesis.so    # kinesis_streamsプラグイン 
...
```

#### ・パブリックECRリポジトリを用いる場合

ECSのコンテナ定義にて、パブリックECRリポジトリのURLを指定し、ECRイメージのプルを実行する。デフォルトで内蔵されているconfファイルの設定をそのまま用いる場合は、こちらを採用する。

参考：https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/firelens-using-fluentbit.html#firelens-image-ecr

#### ・プライベートECRリポジトリを用いる場合

あらかじめ、DockerHubからFluentBitイメージをプルするためのDockerfileを作成し、プライベートECRリポジトリにイメージをプッシュしておく。ECSのコンテナ定義にて、プライベートECRリポジトリのURLを指定し、ECRイメージのプルを実行する。デフォルトで内蔵されているconfファイルの設定を上書きしたい場合は、こちらを採用する。

```dockerfile
FROM amazon/aws-for-fluent-bit:latest
```

参考：

- https://hub.docker.com/r/amazon/aws-for-fluent-bit
- https://github.com/aws/aws-for-fluent-bit
- https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/firelens-using-fluentbit.html#firelens-image-dockerhub

<br>

### 標準設定の上書き

#### ・標準設定ファイルの種類

aws-for-fluent-bitイメージの```/fluent-bit/etc```ディレクトリにはデフォルトで設定ファイルが用意されている。追加設定を実行するファイルはここに配置する。

```bash
[root@<コンテナID>:/fluent-bit/etc]$ ls -la

-rw-r--r-- 1 root root  251 Sep  1 17:57 fluent-bit.conf
-rw-r--r-- 1 root root 1564 Sep 27 02:15 fluent-bit_custom.conf # 追加設定用
-rw-r--r-- 1 root root 4664 Sep  1 18:07 parsers.conf
-rw-r--r-- 1 root root  584 Sep  1 18:07 parsers_ambassador.conf
-rw-r--r-- 1 root root  226 Sep  1 18:07 parsers_cinder.conf
-rw-r--r-- 1 root root 2798 Sep  1 18:07 parsers_extra.conf
-rw-r--r-- 1 root root  240 Sep  1 18:07 parsers_java.conf
-rw-r--r-- 1 root root  845 Sep  1 18:07 parsers_mult.conf
-rw-r--r-- 1 root root  291 Sep 27 02:15 parsers_multiline.conf
-rw-r--r-- 1 root root 2954 Sep  1 18:07 parsers_openstack.conf
-rw-r--r-- 1 root root  579 Sep 27 02:15 stream_processor.conf # 追加設定用
```

FireLensコンテナの```/fluent-bit/etc/fluent-bit.conf```ファイルは以下の通りとなり、ローカルPCでFluentBitコンテナを起動した場合と異なる構成になっていることに注意する。

参考：https://dev.classmethod.jp/articles/check-fluent-bit-conf/

```bash
[INPUT]
    Name tcp
    Listen 127.0.0.1
    Port 8877
    Tag firelens-healthcheck

[INPUT]
    Name forward
    unix_path /var/run/fluent.sock

[INPUT]
    Name forward
    Listen 127.0.0.1
    Port 24224

[FILTER]
    Name record_modifier
    Match *
    Record ecs_cluster sample-test-cluster
    Record ecs_task_arn arn:aws:ecs:ap-northeast-1:123456789012:task/sample-test-cluster/d4efc1a0fdf7441e821a3683836ad69a
    Record ecs_task_definition sample-test-webapp-taskdefinition:15

[OUTPUT]
    Name null
    Match firelens-healthcheck
```

#### ・```fluent-bit_custom.conf```ファイル

FireLensコンテナの```/fluent-bit/etc/fluent-bit.conf```ファイルを、コンテナ定義の```config-file-value```キーで指定し、追加設定を実行する。これにより、FireLensコンテナにINCLUDE文が挿入される。

参考：https://dev.classmethod.jp/articles/check-fluent-bit-conf/

```bash
[INPUT]
    Name tcp
    Listen 127.0.0.1
    Port 8877
    Tag firelens-healthcheck
    
[INPUT]
    Name forward
    unix_path /var/run/fluent.sock
    
[INPUT]
    Name forward
    Listen 127.0.0.1
    Port 24224
    
[FILTER]
    Name record_modifier
    Match *
    Record ecs_cluster prd-foo-ecs-cluster
    Record ecs_task_arn arn:aws:ecs:ap-northeast-1:<アカウントID>:task/prd-foo-ecs-cluster/*****
    Record ecs_task_definition prd-foo-ecs-task-definition:1
    
# INCLUDE文が挿入される。ユーザ定義の設定ファイルが読み込まれる。
@INCLUDE /fluent-bit/etc/fluent-bit_custom.conf

[OUTPUT]
    Name laravel
    Match laravel-firelens*
    
[OUTPUT]
    Name nginx
    Match nginx-firelens*    
```

ちなみに、デフォルトの設定ファイルには、INPUTセクションがすでに定義されているため、```fluent-bit_custom.conf```ファイルではINPUTセクションを定義しなくても問題ない。

参考：https://github.com/aws/aws-for-fluent-bit/blob/mainline/fluent-bit.conf

```bash
[INPUT]
    Name        forward
    Listen      0.0.0.0
    Port        24224

[OUTPUT]
    Name cloudwatch
    Match   **
    region us-east-1
    log_group_name fluent-bit-cloudwatch
    log_stream_prefix from-fluent-bit-
    auto_create_group true
```

#### ・```stream_processor.conf```ファイル

STREAM_TASKセクションにて、ログのタグ付けを定義する。FireLensコンテナのパイプラインでは、『<コンテナ名>-firelens-<タスクID>』という名前でログが処理されている。そのため、Stream Processorでログを抽出するためには、クエリで『```FROM TAG:'*-firelens-*'```』を指定する必要がある。ちなみに、STREAM_TASKセクションでタグ付けされたログは、INPUTセクションから再び処理し直される。

参考：https://aws.amazon.com/jp/blogs/news/under-the-hood-firelens-for-amazon-ecs-tasks/

```bash
<コンテナ名>-firelens-<タスクID>: [*****, {"log"=>"127.0.0.1 -  01/01/2022:0:00:00 +0000 "GET /index.php" 200", "container_id"=>"*****", "container_name"=>"laravel", "source"=>"stderr"}]
```

```bash
# laravelコンテナのログへのタグ付け
[STREAM_TASK]
    Name laravel
    Exec CREATE STREAM laravel WITH (tag='laravel') AS SELECT log FROM TAG:'*-firelens-*' WHERE container_name = 'laravel';

# nginxコンテナのログへのタグ付け
[STREAM_TASK]
    Name nginx
    Exec CREATE STREAM nginx WITH (tag='nginx') AS SELECT log FROM TAG:'*-firelens-*' WHERE container_name = 'nginx';

# 全てのコンテナのログへのタグ付け
[STREAM_TASK]
    Name containers
    Exec CREATE STREAM container WITH (tag='containers') AS SELECT * FROM TAG:'*-firelens-*';
```

```bash
[SERVICE]
    Flush 1
    Grace 30
    Log_Level info
    # ファイルを読み込む
    Parsers_File parsers_multiline.conf
    Streams_File stream_processor.conf
```

#### ・```parsers_multiline.conf```ファイル

MULTILINE_PARSERセクションにて、スタックトレースログの各行の結合を定義する。

参考：https://github.com/aws-samples/amazon-ecs-firelens-examples/blob/mainline/examples/fluent-bit/filter-multiline/README.md

```bash
[MULTILINE_PARSER]
    name          laravel
    type          regex
    flush_timeout 1000
    rule          "start_state"   "/(Dec \d+ \d+\:\d+\:\d+)(.*)/"  "cont"
    rule          "cont"          "/^\s+at.*/"                     "cont"
```

```bash
[SERVICE]
    flush                 1
    log_level             info
    parsers_file          /parsers_multiline.conf
    
[FILTER]
    name                  multiline
    match                 *
    multiline.key_content log
    # ファイルを読み込む。組み込みパーサ（goなど）を用いることも可能。
    multiline.parser      go, laravel
```

<br>

### FireLensコンテナのコンテナ定義

#### ・全体

```bash
[
  {
    "name": "<メインコンテナ名>",
    "image": "<ECRリポジトリのURL>",
    "essential": true,
    "portMappings": [
      {
        "containerPort": 80,
        "hostPort": 80,
        "protocol": "tcp"
      }
    ],
    "logConfiguration": {
      "logDriver": "awsfirelens",
      "options": {
        "Name": "forward"
      }
    }
  },
  {
    # FireLensコンテナ名がlog_routerとなることは固定
    "name": "log_router",
    "image": "<ECRリポジトリのURL>",
    "essential": false,
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": {
        # FireLensコンテナ自体がCloudWatchログにログ出力
        "awslogs-group": "<ロググループ名>",
        "awslogs-region": "<リージョン>",
        "awslogs-stream-prefix": "<プレフィクス>"
      }
    },
    "firelensConfiguration": {
      # FireLensコンテナでFluentBitを稼働させる
      "type": "fluentbit",
      "options": {
        "config-file-type": "file",
        # 設定上書きのため読み込み
        "config-file-value": "/fluent-bit/etc/fluent-bit_custom.conf"
        # ECSの情報をFireLensに送信するかどうか
        "enable-ecs-log-metadata": "true"
      }
    },
    "portMappings": [],
    "memoryReservation": 50,
    "secrets": [
      {
        "name": "DD_API_KEY",
        "valueFrom": "<SSMパラメータで管理する環境変数名>"
      },
      {
        "name": "DD_ENV",
        "valueFrom": "<SSMパラメータで管理する環境変数名>"
      }
    ]
  }
]
```

#### ・name

FireLensコンテナをサイドカーとして構築するために、コンテナ定義を実装する。FireLensコンテナは『log_routeter』とする。

#### ・logConfiguration

参考：https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/userguide/firelens-example-taskdefs.html#firelens-example-forward

| 項目                                          | 説明                                                         |
| --------------------------------------------- | ------------------------------------------------------------ |
| ```type```                                    | メインコンテナからFireLensコンテナにログを送信できるように、ログドライバーのタイプとして『```fluentbit```』を設定する。 |
| ```config-file-type```                        | FluentBitの設定ファイルを読み込むために、```file```とする。  |
| ```config-file-value```                       | ```options```キーにて、ログルーティングを設定できるが、それらは```fluent-bit.conf```ファイルにも設定可能であるため、ルーティングの設定はできるだけ```fluent-bit.conf```ファイルに実装する。FireLensコンテナ自体のログは、CloudWatchログに送信するように設定し、メインコンテナから受信したログは監視ツール（Datadogなど）にルーティングする。 |
| ```enable-ecs-log-metadata```（デフォルトで有効化） | 有効にした場合、Datadogのログコンソールで、例えば以下のようなタグが付けられる。<br>![ecs-meta-data_true](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/ecs-meta-data_true.png)<br>反対に無効にした場合、以下のようなタグが付けられる。<br>![ecs-meta-data_false](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/ecs-meta-data_false.png)<br>参考：https://tech.spacely.co.jp/entry/2020/11/28/173356 |
| ```environment```、```secrets```              | コンテナ内の```fluent-bit.conf```ファイルに変数を出力できるように、コンテナの環境変数に値を定義する。 |
