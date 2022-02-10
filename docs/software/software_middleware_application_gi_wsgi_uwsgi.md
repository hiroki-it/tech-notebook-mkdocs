---
title: 【知見を記録するサイト】uWSGI＠ミドルウェア
description: uWSGI＠ミドルウェアの知見をまとめました．
---

# uWSGI＠ミドルウェア

## はじめに

本サイトにつきまして，以下をご認識のほど宜しくお願いいたします．

参考：https://hiroki-it.github.io/tech-notebook-mkdocs/about.html

<br>

## 01. uWSGIの仕組み

### WSGIとして

参考：https://stackoverflow.com/questions/36475380/what-are-the-advantages-of-connecting-uwsgi-to-nginx-using-the-uwsgi-protocol

![uwsgi](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/uwsgi.png)

<br>

## 02. セットアップ

### インストール

#### ・pip経由

```bash
$ pip install uwsgi
```

<br>

## 03. 設定ファイルの種類

### ```uwsgi.ini```ファイル

#### ・```uwsgi.ini```ファイルとは

uWSGIの起動時の値を設定する．JSON形式やXML形式でも問題ない．

参考：

- https://uwsgijapanese.readthedocs.io/ja/latest/Options.html
- https://qiita.com/11ohina017/items/da2ae5b039257752e558

#### ・起動ログ

起動時に，以下のようなログが出力される．

```bash
[uWSGI] getting INI configuration from /etc/wsgi/wsgi.ini

*** Starting uWSGI 2.0.20 (64bit) on [Fri Feb 4 09:18:22 2022] ***
compiled with version: 10.2.1 20210110 on 31 January 2022 23:57:07
os: Linux-4.19.202 #1 SMP Wed Oct 27 22:52:27 UTC 2021
nodename: customer-flask-pod
machine: x86_64
clock source: unix
detected number of CPU cores: 4
current working directory: /var/www/customer
detected binary path: /usr/local/bin/uwsgi
!!! no internal routing support, rebuild with pcre support !!!
uWSGI running as root, you can use --uid/--gid/--chroot options

*** WARNING: you are running uWSGI as root !!! (use the --uid flag) ***
your memory page size is 4096 bytes
detected max file descriptor number: 1048576
lock engine: pthread robust mutexes
thunder lock: disabled (you can enable it with --thunder-lock)
uWSGI http bound on 0.0.0.0:8080 fd 4
uwsgi socket 0 bound to TCP address 127.0.0.1:40133 (port auto-assigned) fd 3
uWSGI running as root, you can use --uid/--gid/--chroot options

*** WARNING: you are running uWSGI as root !!! (use the --uid flag) ***
Python version: 3.10.2 (main, Jan 29 2022, 02:55:36) [GCC 10.2.1 20210110]

*** Python threads support is disabled. You can enable it with --enable-threads ***
Python main interpreter initialized at 0x55c3167f7230
uWSGI running as root, you can use --uid/--gid/--chroot options

*** WARNING: you are running uWSGI as root !!! (use the --uid flag) ***
your server socket listen backlog is limited to 100 connections
your mercy for graceful operations on workers is 60 seconds
mapped 145808 bytes (142 KB) for 1 cores

*** Operational MODE: single process ***
WSGI app 0 (mountpoint='') ready in 1 seconds on interpreter 0x55c3167f7230 pid: 1 (default app)
uWSGI running as root, you can use --uid/--gid/--chroot options

*** WARNING: you are running uWSGI as root !!! (use the --uid flag) ***

*** uWSGI is running in multiple interpreter mode ***
spawned uWSGI master process (pid: 1)
spawned uWSGI worker 1 (pid: 9, cores: 1)
spawned uWSGI http 1 (pid: 10)
```

<br>

## 04. uwsgiセクション

### uwsgiセクションとは

uWSGIを設定する．

<br>

### callable

 アプリケーションのインスタンスの変数名を設定する．デフォルト値は，```application```である．

参考：https://laplace-daemon.com/nginx-uwsgi-flask/

```ini
[uwsgi]
callable = app
```

<br>

### chdir

作業ディレクトリから移動する

```ini
[uwsgi]
chdir=/var/www/foo
```

<br>

### chmod-socket

UNIXドメインソケットファイルの権限を設定する．

```ini
[uwsgi]
chmod-socket = 666
```

<br>

### die-on-term

```ini
[uwsgi]
die-on-term = true
```

<br>

### http

HTTPプロトコルを用いる場合に，受信するIPアドレスとポート番号を設定する．

```ini
[uwsgi]
http = 0.0.0.0:8080
```

<br>

### logto

ログの出力先を設定する．

```ini
[uwsgi]
logto = /dev/stdout
```

<br>

### master

マスターモードで起動するかどうかを設定する

```ini
[uwsgi]
master = true
```

<br>

### processes

```ini
[uwsgi]
processes = 1
```

<br>

### py-autoreload

```ini
[uwsgi]
py-autoreload = 1
```

<br>

### python-path

アプリケーションのあるディレクトリを設定する．

```ini
[uwsgi]
python-path = /var/www/foo
```

<br>

### socket

UNIXドメインソケットを用いる場合に，ソケットファイルの生成場所と受信するポート番号を設定する．

参考： https://qiita.com/koyoru1214/items/57461b920dfc11f67683

```ini
[uwsgi]
socket = /etc/uwsgi/uwsgi.sock:8080
```

<br>

### vacuum

uwsgiプロセス終了時にソケットファイルを削除するかどうかを設定する．

```ini
[uwsgi]
vacuum = true
```

<br>

### wsgi-file

エントリーポイントとするファイルを設定する．

参考：https://django.kurodigi.com/uwsgi-basic/

```ini
[uwsgi]
wsgi-file = main.py
```