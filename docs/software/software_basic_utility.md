---
title: 【IT技術の知見】ユーティリティ（サービスプログラム）＠基本ソフトウェア
description: ユーティリティ（サービスプログラム）＠基本ソフトウェアの知見を記録しています。
---

# ユーティリティ（サービスプログラム）＠基本ソフトウェア

## はじめに

本サイトにつきまして、以下をご認識のほど宜しくお願いいたします。

ℹ️ 参考：https://hiroki-it.github.io/tech-notebook-mkdocs/about.html

<br>

## 01. ユーティリティの種類

### UNIXの場合

本ノートではUNIXのユーティリティを整理する（RedHat系とDebian系のユーティリティを別々に整理したい...）。

<br>

### Windowsの場合

Windowsは、GUIでユーティリティを使用する。よく使用するものを記載する。

| システム系     | ストレージデバイス管理系 | ファイル管理系     | その他       |
|-----------|--------------|-------------|-----------|
| マネージャ     | デフラグメントツール   | ファイル圧縮プログラム | スクリーンセーバー |
| クリップボード   | アンインストーラー    | -           | ファイアウォール  |
| レジストリクリーナ | -            | -           | -         |
| アンチウイルス   | -            | -           | -         |

<br>

## 02. ユーティリティのバイナリファイルの場所

### ディレクトリとバイナリファイルの種類

| バイナリファイルのディレクトリ       | 配置されているバイナリファイルの種類                                                  |
|-----------------------|---------------------------------------------------------------------|
| ```/bin```            | UNIXユーティリティのバイナリファイルの多く。                                            |
| ```/usr/bin```        | 管理ユーティリティによってインストールされるバイナリファイルの多く。                                  |
| ```/usr/local/bin```  | UNIX外のソフトウェアによってインストールされたバイナリファイル。最初は空になっている。                       |
| ```/sbin```           | UNIXユーティリティのバイナリファイルうち、```sudo```権限が必要なもの。                          |
| ```/usr/sbin```       | 管理ユーティリティによってインストールされたバイナリファイルのうち、```sudo```権限が必要なもの。               |
| ```/usr/local/sbin``` | UNIX外のソフトウェアによってインストールされたバイナリファイルのうち、```sudo```権限が必要なもの。最初は空になっている。 |

### バイナリファイルの場所の探し方

```bash
# バイナリファイルが全ての場所で見つからないエラー
$ which python3
which: no python3 in (/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin)

# バイナリファイルの場所
$ which python3 
/usr/bin/python3
```

<br>

## 03. ユーティリティ

### chmod：change mode

#### ▼ ```<数字>```

ファイルの権限を変更する。よく使用されるパーミッションのパターンは次の通り。

```bash
$ chmod 600 <ファイルへのパス>
```

#### ▼ -R ```<数字>```

ディレクトリ内のファイルに対して、再帰的に権限を付与する。ディレクトリ名にスラッシュをつける必要がある。

ℹ️ 参考：http://raining.bear-life.com/linux/chmod%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%80%81%E3%83%87%E3%82%A3%E3%83%AC%E3%82%AF%E3%83%88%E3%83%AA%E3%81%AE%E3%83%91%E3%83%BC%E3%83%9F%E3%83%83%E3%82%B7%E3%83%A7%E3%83%B3%E5%A4%89%E6%9B%B4

```bash
$ chmod -R 600 <ディレクトリ名>/
```

#### ▼ 100番刻みの規則性

所有者以外に全権限が与えられない。

| 数字 | 所有者 | グループ | その他 | 特徴                   |
| :--: | :----- | :------- | :----- | ---------------------- |
| 500  | r-x    | ---      | ---    | 所有者以外に全権限なし |
| 600  | rw-    | ---      | ---    | 所有者以外に全権限なし |
| 700  | rwx    | ---      | ---    | 所有者以外に全権限なし |

#### ▼ 111番刻みの規則性

全てのパターンで同じ権限になる。

| 数字 | 所有者 | グループ | その他 | 特徴                 |
| :--: | :----- | :------- | :----- | -------------------- |
| 555  | r-x    | r-x      | r-x    | 全てにWrite権限なし  |
| 666  | rw-    | rw-      | rw-    | 全てにExecut権限なし |
| 777  | rwx    | rwx      | rwx    | 全てに全権限あり     |

#### ▼ その他でよく使用する番号

| 数字 | 所有者 | グループ | その他 | 特徴                               |
| :--: | :----- | :------- | :----- | ---------------------------------- |
| 644  | rw-    | r--      | r--    | 所有者以外にWrite、Execute権限なし |
| 755  | rwx    | r-x      | r-x    | 所有者以外にWrite権限なし          |

#### ▼ go

現在の```chmod```コマンドの実行者以外にアクセス権限を付与する。

ℹ️ 参考：http://www.damp.tottori-u.ac.jp/~ooshida/unix/chmod.html

```bash
$ chmod go+r <ファイルへのパス>
```

<br>

### cp

#### ▼ -Rp

ディレクトリの属性情報も含めて、ディレクトリとファイルを再帰的にコピーする。

```bash
$ cp -Rp /<ディレクトリ名1>/<ディレクトリ名2> /<ディレクトリ名1>/<ディレクトリ名2>
```

```bash
# 隠しファイルも含めて、ディレクトリの中身を他のディレクトリ内にコピー
# 『アスタリスク』でなく『ドット』にする
$ cp -Rp /<ディレクトリ名>/ /<ディレクトリ名> 
```

#### ▼ -p

『```<ファイルへのパス>.YYYYmmdd```』の形式でバックアップファイルを作成する。

```bash
$ cp -p <ファイルへのパス> <ファイルへのパス>.`date +"%Y%m%d"`
```

<br>

### Cron

#### ▼ Cronとは

UNIXにて、Linuxの機能の一つであるジョブ管理を実現する。あらかじめ、ジョブ（定期的に実行するように設定されたバッチ処理）を登録しておき、指定したスケジュールに従って、ジョブを実行する。

#### ▼ ```cron```ファイル

```bash
# /etc/cron.hourly/cron-hourly.txt
# バッチ処理を、毎時・1分ごとに実行するように設定する。
1 * * * * root run-parts /etc/cron.hourly
# <- 最後は改行する。
```

| ディレクトリ名          | 利用者 | 主な用途                                         |
| ----------------------- | ------ | ------------------------------------------------ |
| ```/etc/crontab```      | root   | 任意のcronファイルを配置するディレクトリ         |
| ```/etc/cron.hourly```  | root   | 毎時実行されるcronファイルを配置するディレクトリ |
| ```/etc/cron.daily```   | root   | 毎日実行されるcronファイルを配置するディレクトリ |
| ```/etc/cron.monthly``` | root   | 毎月実行されるcronファイルを配置するディレクトリ |
| ```/etc/cron.weekly```  | root   | 毎週実行されるcronファイルを配置するディレクトリ |

**＊実装例＊**

（１）あらかじめ、各ディレクトリにcronファイルを配置しておく。

（２）ジョブを登録するファイルを作成する。```run-parts```コマンドで、指定した時間に、各cronディレクトリ内の```cron```ファイルを一括で実行するように記述しておく。

```bash
# 設定
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO="example@gmail.com"
LANG=ja_JP.UTF-8
LC_ALL=ja_JP.UTF-8
CONTENT_TYPE=text/plain; charset=UTF-8

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed

# ジョブ
1 * * * * root run-parts /etc/cron.hourly # 毎時・1分
5 2 * * * root run-parts /etc/cron.daily # 毎日・2時5分
20 2 * * 0 root run-parts /etc/cron.weekly # 毎週日曜日・2時20分
40 2 1 * * root run-parts /etc/cron.monthly # 毎月一日・2時40分
@reboot make clean html # cron起動時に一度だけ
```


#### ▼ ```cron.d```ファイル

複数の```cron```ファイルで全ての1つのディレクトリで管理する場合に使用する。

| ディレクトリ名           | 利用者  | 主な用途                      |
|-------------------|------|---------------------------|
| ```/etc/cron.d``` | root | 上記以外の自動タスク設定ファイルを配置するディレクトリ |

<br>

### crond

#### ▼ crondとは

cronデーモンを起動するためのプログラム

#### ▼ -n

フォアグラウンドプロセスとしてcronを起動

```bash
$ crond -n
```

<br>

### crontab

#### ▼ crontabとは

crontabファイルを操作するためのユーティリティ。作成した```cron```ファイルを登録できる。```cron.d```ファイルは操作できない。

```bash
$ crontab <ファイルへのパス>
```

**＊実装例＊**

（１）拡張子は自由で、時刻とコマンドが実装されたファイルを用意する。この時、最後に改行がないとエラー（```premature EOF```）になるため、改行を追加する。

ℹ️ 参考：

```bash
# /etc/cron.hourly/cron-hourly.txt
# 毎時・1分
1 * * * * root run-parts /etc/cron.hourly
# <- 最後は改行する。
```

```bash
# /etc/cron.daily/cron-daily.txt
# 毎日・2時5分
5 2 * * * root run-parts /etc/cron.daily
# <- 最後は改行する。                         
```

```bash
# /etc/cron.monthly/cron-monthly.txt
# 毎週日曜日・2時20分
20 2 * * 0 root run-parts /etc/cron.weekly
# <- 最後は改行する。
```

```bash
# /etc/cron.weekly/cron-weekly.txt
# 毎月一日・2時40分
40 2 1 * * root run-parts /etc/cron.monthly
# <- 最後は改行する。
```

```bash
# cron起動時に一度だけ
@reboot make clean html
# <- 最後は改行する。
```

（２）このファイルを```crontab```コマンドで登録する。cronファイルの実体はないことと、ファイルの内容を変更した場合は登録し直さなければいけないことに注意する。

```bash
$ crontab /etc/cron.hourly/cron-hourly.txt
```

（３）登録されている処理を取得する。

```bash
$ crontab -l

1 * * * * root run-parts /etc/cron.hourly/cron.hourly
```

（４）ログに表示されているかを確認する。

```bash
$ cd /var/log

$ tail -f cron
```

（５）改行コードを確認。改行コードが表示されない場合はLFであり、問題ない。

```bash
$ file /etc/cron.hourly/cron-hourly.txt

foo.txt: ASCII text
```

#### ▼ -e

エディタを開き、登録済みのcronファイルを変更/削除する。

ℹ️ 参考：https://nontitle.xyz/archives/1065

```bash
$ crontab -e

# 登録されたcronファイルが表示されるため、変更/削除する。
1 * * * * rm foo
```

#### ▼ -l

登録済みのcronファイルの一覧を表示する。

```bash
$ crontab -l

# crontabコマンドで登録されたcronファイルの処理
1 * * * * rm foo
```

<br>

### curl

#### ▼ curlとは

GETリクエストを送信する。```jq```コマンドを使用すると、レスポンスを整形できる。

**＊実行例＊**

```bash
$ curl -X GET https://example.com/foo/1 | jq . 
```

#### ▼ -d

メッセージボディを設定する。

**＊実行例＊**

```bash
$ curl -X POST -H "Content-Type: application/json" -d '{}' https://example.com/foo
```

#### ▼ -L

指定したURLでリダイレクトが行われても、リダイレクト後のURLからファイルをインストールする。

**＊実行例＊**

```bash
$ curl -L https://example.com/foo
```

#### ▼ -o（小文字）

インストール後のファイル名を定義する。これを指定しない場合、```-O```オプションを有効化する必要がある。

**＊実行例＊**

```bash
$ curl -o <ファイルへのパス> https://example.com
```

#### ▼ -O（大文字）

インストール後のファイル名はそのままでインストールする。これを指定しない場合、```-o```オプションを有効化する必要がある。

#### ▼ --resolve

ドメインとIPアドレスを紐付け、指定した名前解決を行いつつ、```curl```コマンドを実行する。

```bash
$ curl --resolve <ドメイン名>:<ポート番号>:<IPアドレス> https://example.com
```

**＊実行例＊**

リクエストの名前解決時に、```example.com```を正引きすると```127.0.0.1```が返却されるようにする。

```bash
$ curl --resolve example.com:80:127.0.0.1 https://example.com
```

#### ▼ -X

HTTPメソッドを設定する。

**＊実行例＊**

```bash
curl -X GET https://example.com
```

<br>

### df

#### ▼ dfとは

パーティションで区切られたストレージのうち、マウントされているもののみを取得する。

ℹ️ 参考：https://hiroki-it.github.io/tech-notebook-mkdocs/software/software_basic_kernel.html

**＊実行例＊**

```bash
$ df

Filesystem      512-blocks      Used Available Capacity iused      ifree %iused  Mounted on
/dev/disk1s1s1   976490576  44424136 671194960     7%  559993 4881892887    0%   /
devfs                  393       393         0   100%     681          0  100%   /dev
/dev/disk1s5     976490576  10485816 671194960     2%       5 4882452875    0%   /System/Volumes/VM
/dev/disk1s3     976490576    774096 671194960     1%    1648 4882451232    0%   /System/Volumes/Preboot
/dev/disk1s6     976490576    224048 671194960     1%     450 4882452430    0%   /System/Volumes/Update
/dev/disk1s2     976490576 247856952 671194960    27% 1367035 4881085845    0%   /System/Volumes/Data
map auto_home            0         0         0   100%       0          0  100%   /System/Volumes/Data/home
/dev/disk1s1     976490576  44424136 671194960     7%  553759 4881899121    0%   /System/Volumes/Update/mnt1
/dev/disk2s1        610224    421128    189096    70%     360 4294966919    0%   /Volumes/Kindle
/dev/disk3s1        188336    149544     38792    80%     735 4294966544    0%   /Volumes/Amazon Chime

```

#### ▼ -h、-m、-t

ストレージの使用状況をメガバイトで取得する。

```bash
# h：--human-readable
# t：--total
$ df -h -m -t
```

#### ▼ fdiskとの違い

類似する```df```コマンドでは、パーティションで区切られたストレージのうちでマウントされたもののみを取得する。一方で```fdisk```コマンドでは、マウントされているか否かに関わらず、パーティションで区切られた全てのストレージを取得する。

ℹ️ 参考：https://stackoverflow.com/questions/16307484/difference-between-df-h-and-fdisk-command

<br>

### dig

#### ▼ digとは

正引きの名前解決を実行する

ℹ️ 参考：

- https://qiita.com/hypermkt/items/610b5042d290348a9dfa#%E3%83%98%E3%83%83%E3%83%80%E3%83%BC
- https://dev.classmethod.jp/articles/dig-route53-begginer/

```bash
$ dig yahoo.co.jp 

# Header
# 各セクションのステータスやフラグが表示される。
; <<>> DiG 9.10.6 <<>> yahoo.co.jp
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 23877 # 『NOERROR』は、正引きが成功したことを表す。
;; flags: qr rd ra; QUERY: 1, ANSWER: 8, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512

# Questionセクション
;; QUESTION SECTION:
;yahoo.co.jp.                   IN      A # Aレコードを問い合わせたことを表す。

# Answerセクション
# DNSレコード
;; ANSWER SECTION:
yahoo.co.jp.            35      IN      A       182.22.28.252 # 正引きで返却されたIPアドレスを表す。
yahoo.co.jp.            35      IN      A       182.22.16.251
yahoo.co.jp.            35      IN      A       183.79.217.124
yahoo.co.jp.            35      IN      A       183.79.219.252
yahoo.co.jp.            35      IN      A       183.79.250.123
yahoo.co.jp.            35      IN      A       182.22.25.124
yahoo.co.jp.            35      IN      A       183.79.250.251
yahoo.co.jp.            35      IN      A       182.22.25.252

# 正引きにかかった時間を表す。
;; Query time: 7 msec
# 正引きに利用したDNSサーバーを表す。
# digコマンドのパラメーターでDNSサーバーを指定しない場合、digコマンドの実行元によって、異なるDNSサーバーが利用される。
;; SERVER: 8.8.8.8#53(8.8.8.8) 
;; WHEN: Mon May 30 22:33:44 JST 2022
;; MSG SIZE  rcvd: 168

```

#### ▼ -x

逆引きの名前解決を実行する。

ℹ️ 参考：https://atmarkit.itmedia.co.jp/ait/articles/1409/25/news001.html

```bash
$ dig -x 182.22.28.252

; <<>> DiG 9.10.6 <<>> -x 182.22.28.252
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 9847
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;252.28.22.182.in-addr.arpa.    IN      PTR

# AUTHORITYセクション
# 権威DNSサーバーを表す。ドメイン名がわかる。
;; AUTHORITY SECTION:
28.22.182.in-addr.arpa. 663     IN      SOA     yahoo.co.jp. postmaster.yahoo.co.jp. 2202070000 1800 900 1209600 900

;; Query time: 7 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Mon May 30 22:47:07 JST 2022
;; MSG SIZE  rcvd: 113
```

<br>

### du

#### ▼ duとは

指定したディレクトリ内のサブディレクトリのサイズ、ディレクトリ全体の合計サイズ（KB）、を取得する。

```bash
# 表示結果をサイズの降順に並び替える。
$ du ./ | sort -n

21816   ./vendor/foo/bar/baz/qux
27004   ./vendor/foo/bar/baz
27036   ./vendor/foo/bar
27604   ./vendor/foo
115104  ./vendor
123016  ./
```

#### ▼ -h

読みやすい単位で、指定したディレクトリ内のサブディレクトリのサイズ、ディレクトリ全体の合計サイズ（KB）、を取得する。ただし、細かい数値が省略されてしまうため、より正確なサイズを知りたい場合は、```-h```オプションを使用しないようにする。

ℹ️ 参考：https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/6/html/deployment_guide/s2-sysinfo-filesystems-du

```bash
$ du -h ./

21K   ./vendor/foo/bar/baz/qux # 読みやすいが、細かい数値は省略されてしまう。
27K   ./vendor/foo/bar/baz
27K   ./vendor/foo/bar
27K   ./vendor/foo
1.1M  ./vendor
1.2M  ./
```

#### ▼ -s

指定したディレクトリ内の合計サイズ（KB）を取得する。

ℹ️ 参考：https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/6/html/deployment_guide/s2-sysinfo-filesystems-du

```bash
$ du -s ./

12345678 ./
```

<br>

### echo

#### ▼ echoとは

定義されたシェル変数を出力する。変数名には```$```マークを付ける。ダブルクオートはあってもなくても良い。

```bash
$ <変数名>=<値>

$ echo $<変数名>

$ echo "$<変数名>"
```

<br>

### export

#### ▼ exportとは

基本的な手順としては、シェル変数を設定し、これを環境変数に追加する。シェル変数と環境変数については、以下のリンクを参考にせよ。

ℹ️ 参考：https://hiroki-it.github.io/tech-notebook-mkdocs/software/software_basic_utility_shell.html

```bash
# シェル変数を設定
$ PATH=$PATH:<バイナリファイルへのあるディレクトリへの絶対パス>
# 環境変数に追加
$ export PATH
```

シェル変数の設定と、環境変数への追加は、以下の通り同時に記述できる。

```bash
# 環状変数として、指定したバイナリファイル（bin）のあるディレクトリへの絶対パスを追加。
# バイナリファイルを入力すると、絶対パス
$ export PATH=$PATH:<バイナリファイルへのあるディレクトリへの絶対パス>
```

```bash
# 不要なパスを削除したい場合はこちら
# 環状変数として、指定したバイナリファイル（bin）のあるディレクトリへの絶対パスを上書き
$ export PATH=/sbin:/bin:/usr/sbin:/usr/bin
```

#### ▼ ```/home/centos/.bashrc```ファイル

OSを再起動すると、```export```コマンドの結果は消去されてしまう。そのため、再起動時に自動的に実行されるよう、```.bashrc```ファイルに追記しておく。

```bash
# Source global definitions
if [ -f /etc/bashrc ]; then
  . /etc/bashrc
fi

# User specific environment
PATH="$HOME/.local/bin:$HOME/bin:$PATH"

# fooバイナリファイルのパスを追加 を追加 <--- ここに追加
PATH=$PATH:/usr/local/sbin/foo

export PATH

# Uncomment the following line if you don"t like systemctl"s auto-paging feature:
# export SYSTEMD_PAGER=

# User specific aliases and functions
```

<br>

### fdisk

#### ▼ -l

パーティションで区切られた全てのストレージを取得する。

<br>

### file

#### ▼ fileとは

ファイルの改行コードを取得する。

```bash
# LFの場合（何も表示されない）
$ file foo.txt
foo.txt: ASCII text

# CRLFの場合
$ file foo.txt
foo.txt: ASCII text, with CRLF line terminators

# CRの場合
$ file foo.txt
foo.txt: ASCII text, with CR line terminators<br>
```

<br>

### find

#### ▼ -type

ファイルを検索するためのユーティリティ。アスタリスクを付けなくとも、自動的にワイルドカードが働く。

```bash
$ find ./* -type f | xargs grep "<検索文字>"
```

```bash
# パーミッションエラーなどのログを破棄して検索。
$ find ./* -type f | xargs grep "<検索文字>" 2> /dev/null
```

#### ▼ -name

ファイル名が```.conf``` で終わるものを全て検索する。

```bash
$ find ./* -name "*.conf" -type f
```

名前が dir で終わるディレクトリを全て検索する。

```bash
$ find ./* -name "*dir" -type d
```

ルートディレクトリ配下で、 ```<検索文字> ```という文字をもち、ファイル名が```.conf```で終わるファイルを全て検索する。

```bash
$ find ./* -name "*.conf" -type f | xargs grep "<検索文字>"
```

指定した拡張子のファイルを全て削除する。

```bash
$ find ./* -name "*.txt" -type f | xargs rm -rf
```

<br>

### free

#### ▼ -m、--t

物理メモリ、スワップ領域、の使用状況をメガバイトで取得する。

```bash
# m：--mega
# t：--total
$ free -m --t

              total        used        free      shared  buff/cache   available
Mem:          15387        2682        6672           1        6032       12459
Swap:             0           0           0
```

メモリ使用率は、以下の計算式で算出できる。

ℹ️ 参考：https://support.site24x7.com/portal/en/kb/articles/how-is-the-total-memory-utilization-calculated-for-a-linux-server-monitor

```mathematica
メモリ使用率 =
( ( Total - Free ) / Total * 100 ) =
((15387 - 12459) / 15387) * 100 = 19 %
```

<br>

### grep

#### ▼ grepとは

標準出力に出力された文字列のうち、合致するものだけを取得する。文字列の表示に関する様々なユーティリティ（```ls```、```cat```、```find```、など）と組み合わせられる。

```bash
$ cat foo.txt | grep bar
```

```bash
$ cat foo.txt | grep bar
```

#### ▼ -A

標準出力に出力された文字列のうち、以降の数行を取得する。

```bash
$ cat foo.txt | grep -A 5
```

#### ▼ -i

標準出力に出力された文字列のうち、大文字と小文字を区別せずに、合致するものだけを取得する。

```bash
$ cat foo.txt | grep -i bar
```

<br>

### growpart

#### ▼ growpartとは

指定したデバイスファイルに紐づくパーティションを拡張する。

ℹ️ 参考：https://blog.denet.co.jp/try-growpart/

```bash
$ growpart <デバイスファイル名> 1
```

**＊例＊**

先に、```lsblk```コマンドでパーティションを確認する。また、```df```コマンドでパーティションに紐づくデバイスファイルを確認する。

```bash
$ lsblk

NAME          MAJ:MIN RM   SIZE  RO  TYPE  MOUNTPOINT
xvda          202:0    0    16G   0  disk             # ボリューム
└─xvda1       202:1    0     8G   0  part  /          # パーティション
nvme1n1       259:1    0   200G   0  disk  /var/lib

$ df -h

Filesystem     Size   Used  Avail  Use%   Mounted on
/dev/xvda1       8G   1.9G    14G   12%   /           # パーティションに紐づくデバイスファイル
/dev/nvme1n1   200G   161G    40G   81%   /var/lib
```

デバイスファイルを指定し、パーティションを拡張する。

```bash
$ growpart /dev/xvda 1
```

<br>

### history

#### ▼ historyとは

指定した履歴数でコマンドを取得する。

```bash
$ history 100
```

履歴1000件の中からコマンドを検索する。

```bash
$ history 1000 | grep <過去のコマンド>
```

<br>

### ln

#### ▼ シンボリックリンクとは

ファイルやディレクトリのショートカットのこと。シンボリックリンクに対する処理は、リンク先のファイルやディレクトリに転送される。

#### ▼ -s

カレントディレクトリ配下に、シンボリックリンクを作成する。リンクの元になるディレクトリやパスを指定する。

```bash
$ ln -s <リンク先までのパス> <シンボリックリンク名> 
```

<br>

### kill

#### ▼ -9

指定したPIDのプロセスを削除する。

```bash
$ kill -9 <プロセスID>
```

指定したコマンドによるプロセスを全て削除する。

```bash
$ sudo pgrep -f <コマンド名> | sudo xargs kill -9
```

<br>

### logrotate

#### ▼ logrotateとは

ファイルには```2```GBを超えてテキストを書き込めない。そのため、ログを継続的にファイルに書き込む場合は、定期的に、書き込み先を新しいファイルに移行する必要がある。

ℹ️ 参考：

- http://proger.blog10.fc2.com/blog-entry-66.html
- https://milestone-of-se.nesuke.com/sv-basic/linux-basic/logrotate/

<br>

### ls

#### ▼ -1

ファイル名のみを一列で取得する。

```bash
$ ls -1

foo
bar
baz
qux
```

#### ▼ -l、-a

隠しファイルや隠しディレクトリも含めて、全ての詳細を取得する。

```bash
$ ls -l -a

-rw-r--r--  1 root   8238708 Jun 19 20:18 foo
-rw-r--r--  1 root     20734 Jun 19 20:18 bar
-rw-r--r--  1 root 266446929 Jun 19 20:18 baz
-rw-r--r--  1 root 174540990 Jun 19 20:18 qux
```

#### ▼ -h

ファイルサイズをわかりやすい単位で取得する。ディレクトリのサイズは取得できない。

```bash
$ ls -l -h

-rw-r--r-- 1 root root 7.9M Jun 19 20:18 foo
-rw-r--r-- 1 root root 21K  Jun 19 20:18 bar
-rw-r--r-- 1 root root 255M Jun 19 20:18 baz
-rw-r--r-- 1 root root 167M Jun 19 20:18 qux
```

<br>

### lsblk

#### ▼ lsblkとは

ボリュームやパーティション、これに紐づくデバイスファイルのマウントポイント、を取得する。

```bash
$ lsblk

NAME          MAJ:MIN RM   SIZE  RO  TYPE  MOUNTPOINT
xvda          202:0    0    16G   0  disk             # ボリューム
└─xvda1       202:1    0     8G   0  part  /          # パーティション
nvme1n1       259:1    0   200G   0  disk  /var/lib   # ボリューム
```

<br>

### lsof：List open file

#### ▼ -i、-P

使用中のポートをプロセス別に取得する。

```bash
$ lsof -i -P | grep LISTEN

phpstorm   4145 hasegawa   25u  IPv6 *****      0t0  TCP localhost:6942 (LISTEN)
phpstorm   4145 hasegawa   27u  IPv6 *****      0t0  TCP localhost:63342 (LISTEN)
com.docke 46089 hasegawa   63u  IPv6 *****      0t0  TCP *:3500 (LISTEN)
com.docke 46089 hasegawa   75u  IPv4 *****      0t0  TCP localhost:6443 (LISTEN)
com.docke 46089 hasegawa   78u  IPv6 *****      0t0  TCP *:8500 (LISTEN)
com.docke 46089 hasegawa   80u  IPv6 *****      0t0  TCP *:3000 (LISTEN)
LINE      48583 hasegawa    7u  IPv4 *****      0t0  TCP localhost:10400 (LISTEN)
Google    93754 hasegawa  140u  IPv4 *****      0t0  TCP localhost:56772 (LISTEN)
minikube  97246 hasegawa   19u  IPv4 *****      0t0  TCP 192.168.64.1:50252 (LISTEN)
```

<br>

### mkdir

#### ▼ -p

複数階層のディレクトリを作成する。

```bash
$ mkdir -p /<ディレクトリ名1>/<ディレクトリ名2>
```

<br>

### mkswap、swapon、swapoff

#### ▼ スワッピング方式

物理メモリのアドレス空間管理の方法の一種。

![スワッピング方式](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/スワッピング方式.png)

#### ▼ スワップ領域の作成方法

```bash
# 指定したディレクトリをスワップ領域として使用
$ mkswap /swap_volume
```

```bash
# スワップ領域を有効化
# 優先度のプログラムが、メモリからディレクトリに、一時的に退避されるようになる
$ swapon /swap_volume
```

```bash
# スワップ領域の使用状況を確認
$ swapon -s
```

```bash
# スワップ領域を無効化
$ swapoff /swap_volume
```

<br>

### mount

#### ▼ mountとは

指定したデバイスファイルを、これに紐づくディレクトリ（マウントポイント）にマウントする。

ℹ️ 参考：

- https://atmarkit.itmedia.co.jp/ait/articles/1802/15/news035.html
- https://atmarkit.itmedia.co.jp/ait/articles/1802/23/news024.html

```bash
$ mount -t /dev/sdb1 <マウントポイントとなるディレクトリ>
```

#### ▼ -t

マウントのファイル共有システムの種類を設定する。種類によって、パラメータの入力方法が異なる。

ℹ️ 参考：

- https://docs.oracle.com/cd/E19455-01/806-2717/6jbtqleh6/index.html

- https://webkaru.net/linux/mount-command/

NFSによるマウントを実行する。

```bash
$ mount -t nfs <NFSサーバーのホスト名>:<マウント元ディレクトリ> <マウント先ディレクトリ>
```

<br>

### nc：netcat

#### ▼ ncとは

指定したIPアドレス/ドメインに、TCPプロトコルでパケットを送信する。

ℹ️ 参考：https://qiita.com/chenglin/items/70f06e146db19de5a659

```bash
$ nc <IPアドレス/ドメイン> <ポート番号>
```

#### ▼ -v

ログを出力しつつ、```nc```コマンドを実行する。

```bash
$ nc -v <IPアドレス/ドメイン> <ポート番号>
```

パケットに```9000```番ポートに送信する。

```bash
$ nc -v 127.0.0.1 9000

# 失敗の場合
nc: connect to 127.0.0.1 port 9000 (tcp) failed: Connection refused

# 成功の場合
Connection to 127.0.0.1 9000 port [tcp/*] succeeded!
```

<br>

### od：octal dump

#### ▼ odとは

ファイルを8進数の機械語で出力する。

```bash
$ od <ファイルへのパス>
```

#### ▼ -Ad、-tx

ファイルを16進数の機械語で出力する。

```bash
$ od -Ad -tx <ファイルへのパス>
```

<br>

### printenv

#### ▼ printenvとは

全ての環境変数を取得する。

```bash
$ printenv
```

また、特定の環境変数を取得する。

```bash
$ printenv VAR
```

<br>

### ps： process status

#### ▼ -aux

稼働しているプロセスの詳細情報を表示するためのユーティリティ。

```bash
# 稼働しているプロセスのうち、詳細情報に『xxx』を含むものを取得する。
$ ps -aux | grep <検索文字>
```

<br>

### rm

#### ▼ -R

ディレクトリ自体と中のファイルを再帰的に削除する。

```bash
$ rm -R <ディレクトリ名> 
```

<br>

### sed

#### ▼ -i -e ```s/<置換前>/<置換後>/g```

文字列を置換する。また、```-i```オプションで元のファイルを上書きする。```find```コマンドと組み合わせて、特定のファイルのみで実行できるようにすると良い。複数の置換を実行する場合は、```-e```オプションを並べる。

```bash
$ find ./* \
    -name "*.md" \
    -type f | xargs sed -i -e 's/、/、/g' -e 's/。/。/g'
```

スラッシュを含む文字列を置換する場合には、スラッシュをエスケープする必要である。

```bash
$ find ./* \
    -name "*.md" \
    -type f | xargs sed -i -e 's/foo\/bar/FooBar/g'
```

ちなみにMacOSで```-i```オプションを使用する場合は、オプションの引数に空文字を渡す必要がある。

```bash
# MacOSの場合
$ find ./* \
    -name "*.md" \
    -type f | xargs sed -i '' -e 's/foo\/bar/FooBar/g'
```

#### ▼ ```1s/^```

ファイルの一行目にテキストを追加する。

ℹ️ 参考：https://stackoverflow.com/questions/9533679/how-to-insert-a-text-at-the-beginning-of-a-file

```bash
find ./* \
  -name "*.md" \
  -type f | xargs sed -i '1s/^/一行目にFooを挿入して改行\n\n/'
```

```bash
# MacOSの場合
find ./* \
  -name "*.md" \
  -type f | xargs sed -i '' '1s/^/一行目にFooを挿入して改行\n\n/'
```

<br>

### service

アプリケーション系ミドルウェア（PHP-FPM、uWSGI）、Web系ミドルウェア（Apache、Nginx）、データ収集系エージェント（datadogエージェント、cloudwatchエージェント）などで様々なデーモンの操作に使用される。ただし、デーモン自体もコマンドを提供しているため、できる限りデーモンの機能を使用する。

ℹ️ 参考：

- https://hiroki-it.github.io/tech-notebook-mkdocs/software/software_middleware_web_apache_command.html
- https://hiroki-it.github.io/tech-notebook-mkdocs/software/software_middleware_web_nginx_command.html

<br>

### set

#### ▼ setとは

現在設定されているシェル変数の一覧を取得する。

```bash
$ set
```

#### ▼ -n

シェルスクリプトの構文解析を行う。

```bash
$ set -n
```

#### ▼ -e

一連の処理の途中で```0```以外の終了ステータスが出力された場合、全ての処理を終了する。

```bash
$ set -e
```

#### ▼ -x

一連の処理をデバッグ情報として出力する。

```bash
$ set -x
```

#### ▼ -u

一連の処理の中で、未定義の変数が存在した場合、全ての処理を終了する。

```bash
$ set -u
```

#### ▼ -o pipefail

パイプライン（```|```）内の一連の処理の途中で、エラーが発生した場合、その終了ステータスを出力し、全ての処理を終了する。

```bash
$ set -o pipefail
```

<br>

### ssh：secure shell

#### ▼ -l、-p、```<ポート番号>```、-i、-T

事前に、秘密鍵の権限は『```600```』にしておく。tty（擬似ターミナル）を使用する場合は、```-T```オプションをつける。

```bash
$ ssh -l <サーバーのユーザー名>@<サーバーのホスト名> -p 22 -i <秘密鍵のパス> -T
```

#### ▼ -l、-p、```<ポート番号>```、-i、-T、-vvv

```bash
# -vvv：ログを出力する
$ ssh -l <サーバーのユーザー名>@<サーバーのホスト名> -p 22 -i <秘密鍵のパス> -T -vvv
```

#### ▼ ```~/.ssh/config```ファイル

設定が面倒な```ssh```コマンドのオプションの引数を、```~/.ssh/config```ファイルに記述しておく。

```bash
# サーバー１
Host <接続名1>
    User <サーバー１のユーザー名>
    Port 22
    HostName <サーバー１のホスト名>
    IdentityFile <秘密鍵へのパス>

# サーバー２
Host <接続名２>
    User <サーバー２のユーザー名>
    Port 22
    HostName <サーバー２のホスト名>
    IdentityFile <秘密へのパス>
```

これにより、コマンド実行時の値渡しを省略できる。tty（擬似ターミナル）を使用する場合は、-Tオプションをつける。

```bash
# 秘密鍵の権限は、事前に『600』にしておく
$ ssh <接続名> -T
```

<br>

### strace

####  ▼ straceとは

ユーティリティによるシステムコールをトレースする。

```bash
$ strace <任意のユーティリティ>
```

#### ▼ -c

システムコールごとに情報を取得する。

```bash
# curlコマンドのシステムコールをトレースする。
$ strace -c curl -s -o /dev/null https://www.google.com/

% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
 17.15    0.010642          76       139           mmap
 16.64    0.010329         120        86         2 read
 16.64    0.010326         137        75           rt_sigaction

# 〜 中略 〜

  0.08    0.000048          24         2         2 access
  0.00    0.000000           0         1           execve
------ ----------- ----------- --------- --------- ----------------
100.00    0.062070                   584        12 total
```

#### ▼ -e

トレースの内容をフィルタリングし、取得する。

```bash
# ネットワークに関する情報のみを取得する。
$ strace -e trace=network curl -s -o /dev/null https://www.google.com/    

socket(AF_INET6, SOCK_DGRAM, IPPROTO_IP) = 3
socket(AF_INET, SOCK_STREAM, IPPROTO_TCP) = 3
setsockopt(3, SOL_TCP, TCP_NODELAY, [1], 4) = 0

# 〜 中略 〜

sin_addr=inet_addr("*.*.*.*")}, [128->16]) = 0
getsockname(3, {sa_family=AF_INET, sin_port=htons(60714), sin_addr=inet_addr("*.*.*.*")}, [128->16]) = 0

+++ exited with 0 +++
```

#### ▼ -p

ユーティリティがすでに実行途中の場合、プロセスIDを指定してシステムコールをトレースする。

ℹ️ 参考：https://tech-lab.sios.jp/archives/17394

```bash
$ strace -p <プロセスID>
```

<br>

### tar

#### ▼ -x

圧縮ファイルを解凍する。

```bash
$ tar -xf foo.tar.gz
```

#### ▼ -f

圧縮ファイル名を指定する。これを付けない場合、テープドライブが指定される。

```bash
$ tar -xf foo.tar.gz
```

#### ▼ -v

解凍中のディレクトリ/ファイルの作成ログを取得する。

```bash
$ tar -xvf foo.tar.gz

./
./opt/
./opt/foo/
./opt/foo/bar/
./opt/foo/bar/install.sh
./opt/foo/bar/baz/
./opt/foo/bar/baz/init.sh
```

#### ▼ -g

圧縮ファイル（```.gzip```形式）を解凍する。ただし、デフォルトで有効になっているため、オプションは付けないくても問題ない。

```bash
$ tar -zxf foo.tar.gz
```

<br>

### timedatactl

#### ▼ set-timezone

```bash
# タイムゾーンを日本時間に変更
$ timedatectl set-timezone Asia/Tokyo

# タイムゾーンが変更されたかを確認
$ date
```

<br>


### top

#### ▼ topとは

各プロセスの稼働情報（ユーザー名、CPU、メモリ）を取得する。 CPU使用率昇順に並べる

```bash
$ top
```

#### ▼ -a

メモリ使用率昇順に取得する。

```bash
$ top -a
```

<br>

### tr

#### ▼ trとは

指定した文字列をトリミングする。

```bash
#!/bin/bash

cat ./src.txt | tr "\n" "," > ./dst.txt
```

<br>

### tree

#### ▼ treeとは

ディレクトリ構造を取得する。

```bash
$ tree

.
├── foo/
│   └── foo.txt
│
├── bar/
│   └── bar.txt
│
└── baz/
    └── baz.txt
```

#### ▼ -I

パターンにマッチしたファイルを除外し、それ以外のファイルの場合はディレクトリのみを取得する。```-o```オプションで作成されたファイルがある場合に役立つ。

```bash
$ tree -I tree.txt -o tree.txt
```

#### ▼ -o

取得結果をファイルに出力する。

```bash
$ tree -o tree.txt
```

#### ▼ -P

パターンにマッチしたファイルのみを取得し、それ以外のファイルの場合はディレクトリのみを取得する。ディレクトリ内のファイル名にある程度の規則性がある場合に、構造を把握するために役立つ。

```bash
# terraformのproviders.tfファイルのみを取得する。
$ tree -P providers.tf
```

<br>

### traceroute

#### ▼ tracerouteとは

通信の送信元から送信先までに通過するルーターのIPアドレスを取得する。プロトコルやポート番号は指定する必要はない。

![traceroute](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/traceroute.png)

ℹ️ 参考：

- https://webkaru.net/linux/traceroute-command/
- https://faq2.bit-drive.ne.jp/support/traina-faq/result/19-1647?ds=&receptionId=2760&receptionNum=1607536654139&page=1&inquiryWord=&categoryPath=102&selectedDataSourceId=&sort=_score&order=desc&attachedFile=false

```bash
$ traceroute google.com

traceroute to google.com (173.194.38.98), 30 hops max, 60 byte packets
 1  example.com (aaa.bbb.ccc.ddd)  1.016 ms  2.414 ms  2.408 ms
 2  g-o-p-4ee-a01-1-e-1-5.interq.or.jp (210.157.9.233)  0.845 ms  0.861 ms  0.844 ms
 3  g-o-4eb-a13-1-e-2-1.interq.or.jp (210.157.9.209)  0.784 ms  0.786 ms  0.778 ms
 4  b-4ea-b13-1-e-0-1-0.interq.or.jp (210.172.131.149)  2.227 ms  2.218 ms  2.201 ms
 5  b8-e-1-2-0.interq.or.jp (210.172.131.118)  0.658 ms  0.651 ms  0.634 ms
 6  as15169.ix.jpix.ad.jp (210.171.224.96)  2.105 ms  1.433 ms  1.618 ms
 7  209.85.243.58 (209.85.243.58)  1.623 ms  35.993 ms  1.596 ms
 8  209.85.251.239 (209.85.251.239)  2.357 ms  2.595 ms  2.475 ms
 9  nrt19s18-in-f2.1e100.net (173.194.38.98)  1.812 ms  1.849 ms  1.955 ms
```

もし、IPアドレスがアスタリスクになった場合は、それ以降のルーターに通信が届いていないことを表す。

ℹ️ 参考：https://milestone-of-se.nesuke.com/nw-basic/ip/traceroute/

```bash
$ traceroute google.com

traceroute to google.com (173.194.38.98), 30 hops max, 60 byte packets
 1  example.com (aaa.bbb.ccc.ddd)  1.016 ms  2.414 ms  2.408 ms
 2  g-o-p-4ee-a01-1-e-1-5.interq.or.jp (210.157.9.233)  0.845 ms  0.861 ms  0.844 ms
 3  g-o-4eb-a13-1-e-2-1.interq.or.jp (210.157.9.209)  0.784 ms  0.786 ms  0.778 ms
 4  b-4ea-b13-1-e-0-1-0.interq.or.jp (210.172.131.149)  2.227 ms  2.218 ms  # ここまでは届く
 5  *  *  *                                                                 # その後失敗
 6  *  *  *
...
```

#### ▼ -n

IPアドレスの名前解決を実行せずに、IPアドレスをそのまま取得する。

ℹ️ 参考：

- https://webkaru.net/linux/traceroute-command/
- https://faq2.bit-drive.ne.jp/support/traina-faq/result/19-1647?ds=&receptionId=2760&receptionNum=1607536654139&page=1&inquiryWord=&categoryPath=102&selectedDataSourceId=&sort=_score&order=desc&attachedFile=false

```bash
$ traceroute -n google.com

traceroute to google.com (173.194.38.105), 30 hops max, 60 byte packets
 1  157.7.140.2  0.916 ms  1.370 ms  1.663 ms
 2  210.157.9.233  0.633 ms  0.735 ms  0.740 ms
 3  210.157.9.209  0.718 ms  0.722 ms  0.761 ms
 4  210.172.131.149  1.520 ms  1.894 ms  1.892 ms
 5  210.172.131.118  0.652 ms  0.645 ms  0.619 ms
 6  210.171.224.96  1.499 ms  1.705 ms  1.587 ms
 7  209.85.243.58  1.575 ms  1.558 ms  1.557 ms
 8  209.85.251.239  2.383 ms  2.740 ms  2.400 ms
 9  173.194.38.105  2.165 ms  1.719 ms  1.840 ms
```

<br>

### unlink

#### ▼ unlinkとは

カレントディレクトリのシンボリックリンクを削除する。

```bash
$ unlink <シンボリックリンク名>
```

<br>

### vim：Vi Imitaion、Vi Improved  

#### ▼ vimとは

ファイルを開き、編集する。

```bash
$ vim <ファイルへのパス>
```

<br>
