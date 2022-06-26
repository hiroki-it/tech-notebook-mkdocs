---
title: 【知見を記録するサイト】Prometheus＠テレメトリー収集ツール
description: Prometheus＠テレメトリー収集ツールの知見をまとめました。

---

# Prometheus＠テレメトリー収集ツール

## はじめに

本サイトにつきまして、以下をご認識のほど宜しくお願いいたします。

参考：https://hiroki-it.github.io/tech-notebook-mkdocs/about.html

<br>

## 01. Prometheusの仕組み

### 構造

Prometheusは、Retrieval、TSDB、HTTPサーバー、から構成されている。Kubernetesリソースのメトリクスのデータポイントを収集し、分析する。また設定された条件下でアラートを生成し、Alertmanagerに送信する。

参考：

- https://prometheus.io/docs/introduction/overview/
- https://knowledge.sakura.ad.jp/11635/#Prometheus-3

![prometheus_architecture](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/prometheus_architecture.png)

<br>

### Prometheus server

#### ▼ Prometheus serverとは

メトリクスのデータポイントを収集し、管理する。またPromQLに基づいて、データポイントからメトリクスを分析できるようにする。```9090```番ポートで、メトリクスのデータポイントをプルし、またGrafanaのPromQLによるアクセスを待ち受ける。

参考：https://knowledge.sakura.ad.jp/27501/#Prometheus_Server

#### ▼ ローカルストレージ

Prometheusは、ローカルの時系列データベースに、収集した全てのメトリクスを保管する。Prometheusは、収集したメトリクスをデフォルトで```2```時間ごとにブロック化し、```data```ディレクトリ配下に配置する。現在処理中のブロックはメモリ上に保持されており、同時に```/data/wal```ディレクトリにもバックアップとして保存される（ちなみにRDBMSでは、これをジャーナルファイルという）。これにより、Prometheusで障害が起こり、メモリ上のブロックが削除されてしまっても、ブロックを復元できる。

参考：https://prometheus.io/docs/prometheus/latest/storage/#local-storage

```yaml
data/
├── 01BKGV7JC0RY8A6MACW02A2PJD/
│   ├── chunks/
│   │   └── 000001
│   │
│   ├── tombstones
│   ├── index
│   └── meta.json
│
├── chunks_head/
│   └── 000001
│
└── wal # WALによるバックアップ
    ├── 000000002
    └── checkpoint.00000001/
        └── 00000000
```

また、ブロックやWALはワーカーNodeにマウントされるため、ワーカーNodeのストレージサイズに注意する必要がある。収集されるデータポイントの合計サイズを小さくする方法として、収集間隔を長くする、不要なデータポイントの収集をやめる、といった方法がある。

参考：https://engineering.linecorp.com/en/blog/prometheus-container-kubernetes-cluster/

```bash
# ワーカーNode内
$ ls -la /var/lib/kubelet/plugins/kubernetes.io/aws-ebs/mounts/aws/ap-northeast-1a/vol-*****/prometheus-db/

-rw-r--r--  1 ec2-user 2000         0 Jun 24 17:07 00004931
-rw-r--r--  1 ec2-user 2000         0 Jun 24 17:09 00004932
-rw-r--r--  1 ec2-user 2000         0 Jun 24 17:12 00004933

# 〜 中略 〜

drwxrwsr-x  2 ec2-user 2000      4096 Jun 20 18:00 checkpoint.00002873.tmp
drwxrwsr-x  2 ec2-user 2000      4096 Jun 21 02:00 checkpoint.00002898
drwxrwsr-x  2 ec2-user 2000      4096 Jun 21 04:00 checkpoint.00002911.tmp
```

#### ▼ リモートストレージ

![prometheus_remote-storage](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/prometheus_remote-storage.png)

Prometheusは、ローカルストレージにメトリクスを保管する代わりに、時系列データベースとして機能するリモートストレージ（AWS Timestream、Google Bigquery、VictoriaMetrics、...）に保管できる。エンドポイントは、『```https://<IPアドレス>/api/v1/write```』になる。Prometheusと外部の時系列データベースの両方を冗長化する場合、冗長化されたPrometheusでは、片方のデータベースのみに送信しないと、メトリクスが重複してしまう

参考：

- https://prometheus.io/docs/prometheus/latest/storage/#remote-storage-integrations
- https://prometheus.io/docs/operating/integrations/#remote-endpoints-and-storage

#### ▼ ダイナミックキュー

リモートストレージにメトリクスを送信する場合に、送信されたメトリクスをキューイングする。ダイナミックキューは、メトリクスのスループットの高さに応じて、キューイングの実行単位であるシャードを増減させる。

参考：https://speakerdeck.com/inletorder/monitoring-platform-with-victoria-metrics?slide=52

![prometheus_dynamic-queues_shard](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/prometheus_dynamic-queues_shard.png)

<br>

### Alertmanager

#### ▼ Alertmanagerとは

Prometheusからアラートを受信し、特定の条件下でルーティングする。

参考：

- https://prometheus.io/docs/alerting/latest/alertmanager/
- https://www.designet.co.jp/ossinfo/alertmanager/
- https://knowledge.sakura.ad.jp/11635/#Prometheus-3

![alertmanager](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/alertmanager.png)

<br>

### Exporter

#### ▼ Exporterとは

PrometheusがPull型通信でメトリクスのデータポイントを収集するためのエンドポイントとして機能する。収集したいメトリクスに合わせて、Exporterを選ぶ必要がある。また、各Exporterは待ち受けているエンドポイントやポート番号が異なっており、Prometheusが各Exporterにリクエストを送信できるように、各ワーカーNodeでエンドポイントやポート番号へのインバウンド通信を許可する必要がある。

参考：https://openstandia.jp/oss_info/prometheus

#### ▼ Exporterタイプ

| タイプ       | 設置方法                                          |
| ------------ | ------------------------------------------------- |
| DaemonSet型  | 各ワーカーNode内に、1つずつ設置する。             |
| Deployment型 | 各ワーカーNode内のDeploymentに、1つずつ設置する。 |
| Sidecar型    | 各ワーカーNode内のPodに、1つずつ設置する。        |

**＊例＊**

参考：

- https://atmarkit.itmedia.co.jp/ait/articles/2205/31/news011.html#072
- https://prometheus.io/docs/instrumenting/exporters/

| Exporter名                                                   | Exportタイプ | ポート番号 | エンドポイント | 説明                                                         |
| :----------------------------------------------------------- | ------------ | ---------- | -------------- | ------------------------------------------------------------ |
| [node-exporter](https://github.com/prometheus/node_exporter) | DaemonSet型  | ```9100``` | ```/metrics``` | ノードのメトリクスのデータポイントを収集する。               |
| [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics) | Deplyoment型 | ```8080``` | 同上           | Kubernetesのリソース単位でメトリクスのデータポイントを収集する。<br>参考：https://tech-blog.abeja.asia/entry/2016/12/20/202631 |
| [nginx-vts-exporter](https://github.com/hnlq715/nginx-vts-exporter) | Sidecar型    | ```9113``` | 同上           | Nginxのメトリクスのデータポイントを収集する。                |
| [apache-exporter](https://github.com/Lusitaniae/apache_exporter) | Sidecar型    | ```9117``` | 同上           | Apacheのメトリクスのデータポイントを収集する。               |
| [black box expoter](https://github.com/prometheus/blackbox_exporter) | Deplyoment型 | ```9115``` | 同上           | 各種通信プロトコルの状況をメトリクスとして収集する。         |
| [mysqld-exporter](https://github.com/prometheus/mysqld_exporter) | Sidecar型    | ```9104``` | 同上           | MySQL/MariaDBのメトリクスのデータポイントを収集する。        |
| [postgres-exporter](https://github.com/prometheus-community/postgres_exporter) | Sidecar型    | ```9187``` | 同上           | PostgreSQLのメトリクスのデータポイントを収集する。           |
| [oracledb-exporter](https://github.com/iamseth/oracledb_exporter) | Sidecar型    | ```9121``` | 同上           | Oracleのメトリクスのデータポイントを収集する。               |
| [elasticsearch-exporter](https://github.com/prometheus-community/elasticsearch_exporter) | Deployment型 | ```9114``` | 同上           | ElasticSearchのメトリクスのデータポイントを収集する。        |
| [redis-exporter](https://github.com/oliver006/redis_exporter) | Sidecar型    | ```9121``` | 同上           | Redisのメトリクスのデータポイントを収集する。                |

#### ▼ PushGateway

PrometheusがPush型メトリクスを対象から収集するためのエンドポイントとして機能する。

参考：https://prometheus.io/docs/practices/pushing/

<br>

## 02. PromQL

### PromQLとは

Prometheusで収集したメトリクスを抽出し、集計できる。

<br>

### データ型

#### ▼ Instant vector

特定の時点の時系列データのこと。

参考：https://it-engineer.hateblo.jp/entry/2019/01/19/150849

#### ▼ Range vector

特定の期間の時系列データのこと。

参考：https://it-engineer.hateblo.jp/entry/2019/01/19/150849

#### ▼ Scalar

浮動小数点の数値型データのこと。

参考：https://it-engineer.hateblo.jp/entry/2019/01/19/150849

#### ▼ String

文字列型データのこと。

参考：https://it-engineer.hateblo.jp/entry/2019/01/19/150849

<br>

### 関数

#### ▼ count

期間内の合計数を算出する。

参考：https://www.opsramp.com/prometheus-monitoring/promql/

#### ▼ increase

rate関数のラッパーであり、rate関数の結果（1秒当たりの平均増加率）に、期間を自動的に掛けた数値（期間あたりの増加数）を算出する。

参考：https://promlabs.com/blog/2021/01/29/how-exactly-does-promql-calculate-rates

```bash
# rate関数に期間（今回は5m）を自動的に掛けた数値を算出する。
increase(foo_metrics[5m])
= rate(foo_metrics[1h]) * 5 * 60
```

#### ▼ rate

平均増加率（%/秒）を算出する。常に同じ割合で増加していく場合、横一直線のグラフになる。

参考：https://www.opsramp.com/prometheus-monitoring/promql/

<br>

## 02-02. メトリクス

### ```prometheus_tsdb_*```

#### ▼ prometheus_tsdb_head_samples_appended_total

Prometheusが収集したデータポイントの合計数を表す。

参考：

- https://valyala.medium.com/prometheus-storage-technical-terms-for-humans-4ab4de6c3d48
- https://christina04.hatenablog.com/entry/prometheus-node-exporter


#### ▼ prometheus_tsdb_compaction_chunk_size_bytes_sum

Prometheusが作成したチャンクの合計サイズ（KB）を表す。

参考：

- https://valyala.medium.com/prometheus-storage-technical-terms-for-humans-4ab4de6c3d48
- https://christina04.hatenablog.com/entry/prometheus-node-exporter



#### ▼ prometheus_tsdb_compaction_chunk_samples_sum

Prometheusが作成したチャンクの合計数を表す。

参考：

- https://valyala.medium.com/prometheus-storage-technical-terms-for-humans-4ab4de6c3d48
- https://christina04.hatenablog.com/entry/prometheus-node-exporter

<br>

## 02-03. クエリのTips

### データポイントの各種数値の算出

#### ▼ データポイントの平均サイズ（KB/秒）の増加率

Prometheusで収集されたデータポイントの平均サイズ（KB/秒）の増加率を分析する。

```bash
rate(prometheus_tsdb_compaction_chunk_size_bytes_sum[1h]) /
rate(prometheus_tsdb_compaction_chunk_samples_sum[1h])
```

#### ▼ データポイントの合計数（個/秒）の増加率

Prometheusで収集されたデータポイントの合計数（個/秒）の増加率を分析する。

```bash
rate(prometheus_tsdb_head_samples_appended_total[1h])
```

#### ▼ データポイントの合計サイズ（KB/秒）の増加率

Prometheusで収集されたデータポイントの合計サイズ（KB/秒）の増加率を分析する。計算式からもわかるように、データポイントの収集の間隔を長くすることで、データポイント数が減るため、合計のサイズを小さくできる。

参考：https://engineering.linecorp.com/en/blog/prometheus-container-kubernetes-cluster/

```bash
rate(prometheus_tsdb_compaction_chunk_size_bytes_sum[1h]) /
rate(prometheus_tsdb_compaction_chunk_samples_sum[1h]) *
rate(prometheus_tsdb_head_samples_appended_total[1h])
```

#### ▼ データポイントの合計サイズ（KB/日）の推移

Prometheusで収集されたデータポイントの合計サイズ（KB/日）の推移を分析する。

```bash
rate(prometheus_tsdb_compaction_chunk_size_bytes_sum[1h]) /
rate(prometheus_tsdb_compaction_chunk_samples_sum[1h]) *
rate(prometheus_tsdb_head_samples_appended_total[1h]) *
60 * 60 * 24
```

<br>

### ストレージの各種数値の算出

#### ▼ ローカルストレージの必要サイズ（KB/日）

データポイントの合計サイズ（KB/日）とローカルストレージの部品ファイルの合計を分析する。ローカルストレージの部品ファイル分で、```20```%のサイズが必要になる。この結果から、ローカルストレージの必要サイズを推測できる。

```bash
rate(prometheus_tsdb_compaction_chunk_size_bytes_sum[1h]) /
rate(prometheus_tsdb_compaction_chunk_samples_sum[1h]) *
rate(prometheus_tsdb_head_samples_appended_total[1h]) *
60 * 60 * 24 *
1.2
```

参考：

- https://www.robustperception.io/how-much-disk-space-do-prometheus-blocks-use/
- https://www.robustperception.io/how-much-space-does-the-wal-take-up/
- https://discuss.prometheus.io/t/prometheus-storage-requirements/268/4
- https://gist.github.com/mikejoh/c172b2400909d33c37199c9114df61ef

#### ▼ リモートストレージの必要サイズ（KB/日）

Prometheusで収集されたデータポイントの全サイズうち、リモートストレージに実際に送信しているサイズ（KB/日）を分析する。この結果から、リモートストレージの必要サイズを推測できる。なお、リモートストレージが送信された全てのデータを保管できるとは限らないため、リモートストレージ側で必要サイズを確認する方がより正確である。

```bash
rate(prometheus_remote_storage_bytes_total[1h]) *
60 * 60 * 24
```