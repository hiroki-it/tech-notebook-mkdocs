---
title: 【知見を記録するサイト】ソフトウェア
---

# ソフトウェア

## はじめに

本サイトにつきまして，以下をご認識のほど宜しくお願いいたします．

参考：https://hiroki-it.github.io/tech-notebook-mkdocs/about.html

<br>

## 01. ソフトウェア

### ソフトウェアとは

システムのうちで，『OS』『ミドルウェア』『アプリケーション（アプリケーションソフトウェア）』の要素のこと．『OS』『ミドルウェア』『ハードウェア』をインフラとも呼ぶ．

参考：https://thinkit.co.jp/article/11526

![software](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/software.png)

<br>

### Twelve-Factor

#### ・Twelve-Factorとは

Webシステムのソフトウェアを開発する上でのベストプラクティスのこと．

参考：https://12factor.net/ja/

<br>

## 02. アプリケーションソフトウェア（応用ソフトウェア）

### アプリケーションソフトウェアの一覧

|                        | ネイティブアプリ | Webアプリとクラウドアプリ | ハイブリッドアプリ |
| :--------------------: | :--------------: | :-----------------------: | :----------------: |
| **利用可能な通信状況** |     On/Off      |            On             |      On/Off       |

<br>

### ネイティブアプリケーション

![ネイティブアプリ](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/ネイティブアプリ.png)

端末のシステムによって稼働するアプリのこと．一度ダウンロードしてしまえば，インターネットに繋がっていなくとも，使用できる．

参考：https://www.sbbit.jp/article/cont1/28197

**＊例＊**

Office，BookLiveのアプリ版

<br>

### Webアプリケーションとクラウドアプリケーション

![Webアプリ](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/Webアプリ.png)

#### ・Webアプリケーション

Webサーバー上で稼働するソフトウェアのこと．URLをWebサーバーにリクエストすることで利用でき，随時，Webサーバーとデータ通信を行う．全ての人が無料で利用できるものと，お金を払った人だけが利用できるものがある．

参考：https://www.sbbit.jp/article/cont1/28197

**＊例＊**

Googleアプリ，Amazon，BookLiveのブラウザ版，サイボウズ

#### ・クラウドアプリケーション

Webサーバー上のソフトウェアによって稼働するアプリのうち，クラウドサービスを提供するもののこと．

**＊例＊**

Google Drive，Dropbox

<br>

### ハイブリッドアプリケーション

![Webviewよるアプリパッケージ](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/Webviewよるアプリパッケージ.png)

![ハイブリッドアプリ](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/ハイブリッドアプリ.png)

端末でWebviewを稼働させ，ソフトウェアのHTLMファイルのレンダリングをWebview上で行うアプリのこと．

参考：https://www.sbbit.jp/article/cont1/28197

**＊例＊**

クックパッド

<br>

## 03. ミドルウェア

### Webサーバーのミドルウェア

- Apache
- Nginx
- IIS

参考：https://thinkit.co.jp/article/11837

<br>

### Appサーバーのミドルウェア

- Apacheの拡張モジュール
- PHP-FPM
- NGINX Unit（WebサーバーのNginxと組み合わせて使用できるミドルウェア）
- Tomcat

参考：https://thinkit.co.jp/article/11837

<br>

### DBサーバーのミドルウェア

- MySQL
- MariaDB
- PostgreSQL
- Oracle Database

参考：https://thinkit.co.jp/article/11837

<br>

## 04. 基本ソフトウェア（広義のOS）

### 基本ソフトウェアの種類

#### ・Unix系OS

Unixを源流として派生したOS．現在では主に，Linux系統（緑色），BSD系統（黄色），SystemV系統（青色）の三つに分けられる．

※ちなみに，MacOSはBSD系統

![Unix系OSの歴史](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/Unix系OSの歴史.png)

#### ・WindowsOS

MS-DOSを源流として派生したOS．今では，全ての派生がWindows 10に集約された．

![WindowsOSの歴史](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/WindowsOSの歴史.png)

<br>

### 基本ソフトウェア

参考：http://kccn.konan-u.ac.jp/information/cs/cyber06/cy6_os.htm

![基本ソフトウェアの構成](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/基本ソフトウェアの構成.png)

<br>

## 04-02. Unix系OS

### Linux系統

#### ・OSとバージョンの確認コマンド

OSとバージョンが```/etc/issue```ファイルに記載されている．

```bash
$ cat /etc/issue
Debian GNU/Linux 10 \n \l
```

現在，Linux系統のOSは，さらに3つの系統に分けられる．

#### ・RedHat系統

RedHat，CentOS，Fedora

#### ・Debian系統

Debian，Ubuntu，

#### ・Slackware系統

Slackware

![Linux distribution](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/LinuxDistribution.png)

<br>

### BSD系統｜MacOS

#### ・環境変数の確認

環境変数を確認する．全ての項目は，実際に実行して確認すること．

```bash
$ export -p

export EDITOR=vim
export HOME=/Users/h.hasegawa
export LANG=en_US.UTF-8
export SHELL=/bin/zsh
export USER=h.hasegawa
export VISUAL=vim
...
```

<br>

## 05. デバイスドライバ

### デバイスドライバとは

要勉強

<br>

## 06. Firmware

### Firmwareとは

ソフトウェア（ミドルウェア ＋ 基本ソフトウェア）とハードウェアの間の段階にあるソフトウェア．ROMに組み込まれている．

### BIOS：Basic Input/Output System

![BIOS](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/BIOS.jpg)

<br>

### UEFI：United Extensible Firmware Interface

Windows 8以降で採用されている新しいFirmware

![UEFIとセキュアブート](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/UEFIとセキュアブート.jpg)

<br>

## 07. OSS：Open Source Software

### OSSとは

以下の条件を満たすソフトウェアをOSSと呼ぶ．アプリケーションソフトウェアから基本ソフトウェアまで，様々なものがある．

1. 利用者は，無償あるいは有償で自由に再配布できる．

2. 利用者は，ソースコードを入手できる．

3. 利用者は，コードを自由に変更できる．また，変更後に提供する場合，異なるライセンスを追加できる．

4. 差分情報の配布を認める場合には，同一性の保持を要求してもかまわない． ⇒ よくわからない

5. 提供者は，特定の個人やグループを差別できない．

6. 提供者は，特定の分野を差別できない．

7. 提供者は，全く同じOSSの再配布でライセンスを追加できない．

8. 提供者は，特定の製品でのみ有効なライセンスを追加できない．

9. 提供者は，他のソフトウェアを制限するライセンスを追加できない．

10. 提供者は，技術的に偏りのあるライセンスを追加できない．

    

### 種類

参考：https://openstandia.jp/oss_info/

![OSS一覧](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/OSS一覧.png)