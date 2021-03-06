---
title: 【IT技術の知見】JMeter＠システムテスト
description: JMeter＠システムテストの知見を記録しています。
---

# JMeter＠システムテスト

## はじめに

本サイトにつきまして、以下をご認識のほど宜しくお願いいたします。

ℹ️ 参考：https://hiroki-it.github.io/tech-notebook-mkdocs/about.html

<br>

## 01. セットアップ

JMeterをインストールし、環境を作成する。

ℹ️ 参考：https://jmeter.apache.org/download_jmeter.cgi

<br>

## 02. JMeterの仕組み

JMeterは、以下のコンポーネントから構成されている。

ℹ️ 参考：https://www.guru99.com/jmeter-element-reference.html

![jmeter_architecuture](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/jmeter_architecuture.png)

<br>

## 03. ユースケース

### ロードテスト

以下の手順で、JMeterを使用したロードテストを実施する。

（１）URLのアクセスランキングを元に、リクエストを送信するためのURLリストを```csv```ファイルで作成する。Googleを参考にしたが、ALBアクセスログを参考にした方が、より正確かもしれない。

（２）JMeterのGUI版にて、シナリオ（```jmx```ファイル）を作成する。スループットコントローラーでURLリスト（```csv```ファイル）をJMeterの組み込み関数で読み込むようにする。csvファイルのリストからランダムに読み出したい場合は、Random関数が適している。スレッド数が例えば```10000```個といった高負荷であると、ローカルマシンがフリーズするため注意すること。

ℹ️ 参考：https://jmeter.apache.org/usermanual/functions.html#__Random

（３）AWSリソースのスペック、VPNなど、再現テストの周辺準備が整っていることを確認する。

（４）JMeterのCUI版のバイナリファイルに、```jmx```ファイルをドラッグ＆ドロップし、テストを実施する。または、バイナリファイルへのパスを通した上で、以下のコマンドでも実行できる。


```bash
$ jmeter -n \
  -t <JMXファイルへのパス> \
  -l <Resultファイルへのパス> \
  -e \
  -o <レポートファイルへのパス>
```
ここで、GUI版を使用しない理由は、コマンドの結果に表示される説明より、GUI版では正しい結果を得られないとのこと、のためである。

```bash
# コマンドの結果
Don't use GUI mode for load testing !, only for Test creation and Test debugging.For load testing, use CLI Mode (was NON GUI):
```

（５）テストを開始後に、結果（```jtl```ファイル）とログ（```log```ファイル）が作成され、テストが終わるまで追記されていく。

（６）テストを修正して新しく実行したい場合、```jmx```ファイル、```jtl```ファイル、logファイルをコピーして、バックアアップしておく。

（７）JMeterのGUI版にて、スレッドグループに、結果をツリーで表示、結果を表で表示、のリスナーを追加する。これらの画面で、```jtl```ファイルを読み込むと、```jtl```ファイルを元にした集計データを得られる。

（８）各種のCloudWatchメトリクスにて、テスト時間帯に着目し、プロットから、数値を読み取る。
