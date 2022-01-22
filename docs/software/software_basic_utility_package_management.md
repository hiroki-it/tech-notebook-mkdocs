---
title: 【知見を記録するサイト】管理ユーティリティ@OS
---

# 管理ユーティリティ@OS

## はじめに

本サイトにつきまして，以下をご認識のほど宜しくお願いいたします．

参考：https://hiroki-it.github.io/tech-notebook-mkdocs/about.html

<br>

##  01. 管理ユーティリティの種類

### 様々な管理ユーティリティ

様々な粒度のプログラムを対象にした管理ユーティリティがある．

![library_package_module](https://raw.githubusercontent.com/hiroki-it/tech-notebook/master/images/library_package_module.png)

<br>

### ライブラリ管理ユーティリティ

| ユーティリティ名                  | 対象プログラミング言語 |
| --------------------------------- | ---------------------- |
| composer.phar：Composer           | PHP                    |
| pip：Package Installer for Python | Python                 |
| npm：Node Package Manager         | Node.js                |
| maven：Apache Maven               | Java                   |
| gem：Ruby Gems                    | Ruby                   |

<br>

### パッケージ管理ユーティリティ

| ユーティリティ名                                       | 対象OS       | 依存関係のインストール可否 |
| ------------------------------------------------------ | ------------ | -------------------------- |
| Rpm：Red Hat Package Manager                           | RedHat系     | ✕                          |
| Yum：Yellow dog Updater Modified<br>DNF：Dandified Yum | RedHat系     | 〇                         |
| Apt：Advanced Packaging Tool                           | Debian系     | 〇                         |
| Apk：Alpine Linux package management                   | Alpine Linux | 〇                         |

<br>

### 言語バージョン管理ユーティリティ

| ユーティリティ名 | 対象プログラミング言語 |
| ---------------- | ---------------------- |
| phpenv           | PHP                    |
| pyenv            | Python                 |
| rbenv            | Ruby                   |

<br>

## 02. ライブラリ管理ユーティリティ

### npm（Node.js）

#### ・入手方法

```bash
# リポジトリの作成
$ curl -sL https://rpm.nodesource.com/setup_<バージョン>.x | bash -

# nodejsのインストールにnpmも含まれる
$ yum install nodejs
```

#### ・init

package.jsonを生成する．

```bash
$ npm init
```

#### ・install

ディレクトリにパッケージをインストール

```bash
# ローカルディレクトリにパッケージをインストール
$ npm install <パッケージ名>
```

```bash
# グローバルディレクトリにインストール（あまり用いない）
$ npm install -g <パッケージ名>
```

<br>

### pip（Python）

#### ・install

指定したライブラリをインストールする．

参考：https://pip-python3.readthedocs.io/en/latest/reference/pip_install.html#pip-install

```bash
# /usr/local 以下にインストール
$ pip install --user <ライブラリ名>
```

```bash
# requirements.txt を元にライブラリをインストール
$ pip install -r requirements.txt

# 指定したディレクトリにライブラリをインストール
pip install -r requirements.txt　--prefix=/usr/local
```

#### ・freeze

pipでインストールされたパッケージを元に，要件ファイルを作成する．

参考：https://pip-python3.readthedocs.io/en/latest/reference/pip_freeze.html

```bash
# インストールのため
$ pip freeze > requirements.txt

# アンインストールのため
$ pip freeze > uninstall.txt
```

#### ・show

pipでインストールしたパッケージ情報を表示する．

参考：https://pip-python3.readthedocs.io/en/latest/reference/pip_show.html

```bash
$ pip show sphinx

Name: Sphinx
Version: 3.2.1
Summary: Python documentation generator
Home-page: http://sphinx-doc.org/
Author: Georg Brandl
Author-email: georg@python.org
License: BSD
# インストール場所
Location: /usr/local/lib/python3.8/site-packages
# このパッケージの依存対象
Requires: sphinxcontrib-applehelp, imagesize, docutils, sphinxcontrib-serializinghtml, snowballstemmer, sphinxcontrib-htmlhelp, sphinxcontrib-devhelp, sphinxcontrib-jsmath, setuptools, packaging, Pygments, babel, alabaster, sphinxcontrib-qthelp, requests, Jinja2
# このパッケージを依存対象としているパッケージ
Required-by: sphinxcontrib.sqltable, sphinx-rtd-theme, recommonmark
```

#### ・uninstall

指定したライブラリをインストールする．

参考：https://pip-python3.readthedocs.io/en/latest/reference/pip_uninstall.html

```bash
$ pip uninstall -y <ライブラリ名>
```

```bash
# uninstall.txt を元にライブラリをアンインストール
$ pip uninstall -y -r uninstall.txt
```

<br>

## 03. パッケージ管理ユーティリティ

### apt-file（Debian系）

#### ・search

指定したファイルを持つパッケージを検索する．拡張子も指定しても，ファイル名までしか絞れない．

参考：

- https://atmarkit.itmedia.co.jp/ait/articles/1709/08/news020.html
- https://embedded.hatenadiary.org/entry/20081101/p3

```bash
# apt-fileパッケージをインストールする．
$ apt-get install apt-file

$ apt-file update

# zlib.hファイルを持つパッケージを検索する．
$ apt-file search zlib.h

autoconf-archive: /usr/share/doc/autoconf-archive/html/ax_005fcheck_005fzlib.html
cc65: /usr/share/cc65/include/zlib.h
dovecot-dev: /usr/include/dovecot/istream-zlib.h
dovecot-dev: /usr/include/dovecot/ostream-zlib.h

# ～ 中略 ～

tcllib: /usr/share/doc/tcllib/html/tcllib_zlib.html
texlive-plain-generic: /usr/share/texlive/texmf-dist/tex4ht/ht-fonts/alias/arabi/nazlib.htf
tinc: /usr/share/doc/tinc/tinc.html/zlib.html
zlib1g-dev: /usr/include/zlib.h
```

<br>

### rpm（RedHat系）

#### ・-ivh

パッケージをインストールまたは更新する．一度に複数のオプションを組み合わせて記述する．インストール時にパッケージ間の依存関係を解決できないので注意．

```bash
# パッケージをインストール
# -ivh：--install -v --hash 
$ rpm -ivh <パッケージ名>
```

```bash
# パッケージを更新
# -Uvh：--upgrade -v --hash 
$ rpm -Uvh <パッケージ名>
```

#### ・-qa

インストールされた全てのパッケージの中で，指定した文字を名前に含むものを表示する．

```bash
# -qa：
$ rpm -qa | grep <検索文字>
```

#### ・-ql

指定したパッケージ名で，関連する全てのファイルの場所を表示する．

```bash
# -ql：
$ rpm -ql <パッケージ名>
```

#### ・-qi

指定したパッケージ名で，インストール日などの情報を表示する．

```bash
# -qi：
$ rpm -qi <パッケージ名>
```

<br>

### yum，dnf（RedHat系）

#### ・install，reinstall

rpmと同様の使い方ができる．また，インストール時にパッケージ間の依存関係を解決できる．

```bash
# パッケージをインストール
$ yum install -y <パッケージ名>

# 再インストールする時は，reinstallとすること
$ yum reinstall -y <パッケージ名>
```

#### ・list

インストールされた全てのパッケージを表示する．

```bash
# 指定した文字を名前に含むものを表示．
$ yum list | grep <検索文字>
```

#### ・EPELリポジトリ，Remiリポジトリ

CentOS公式リポジトリはパッケージのバージョンが古いことがある．そこで，```--enablerepo```オプションを用いると，CentOS公式リポジトリではなく，最新バージョンを扱う外部リポジトリ（EPEL，Remi）から，パッケージをインストールできる．外部リポジトリ間で依存関係にあるため，両方のリポジトリをインストールする必要がある．

1. CentOSのEPELリポジトリをインストール．インストール時の設定ファイルは，/etc/yu.repos.d/* に配置される．

```bash
# CentOS7系の場合
$ yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

# CentOS8系の場合
$ dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

# こちらでもよい
$ yum install -y epel-release でもよい
```

2. CentOSのRemiリポジトリをインストール．RemiバージョンはCentOSバージョンを要確認．インストール時の設定ファイルは，```/etc/yu.repos.d/*```に配置される．

```bash
# CentOS7系の場合
$ yum install -y https://rpms.remirepo.net/enterprise/remi-release-7.rpm

# CentOS8系の場合
$ dnf install -y https://rpms.remirepo.net/enterprise/remi-release-8.rpm
```

4. 設定ファイルへは，インストール先のリンクなどが自動的に書き込まれる．

```bash
[epel]
name=Extra Packages for Enterprise Linux 6 - $basearch
#baseurl=http://download.fedoraproject.org/pub/epel/6/$basearch
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-6&arch=$basearch
failovermethod=priority
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6

[epel-debuginfo]
name=Extra Packages for Enterprise Linux 6 - $basearch - Debug
#baseurl=http://download.fedoraproject.org/pub/epel/6/$basearch/debug
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-debug-6&arch=$basearch
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
gpgcheck=1

[epel-source]
name=Extra Packages for Enterprise Linux 6 - $basearch - Source
#baseurl=http://download.fedoraproject.org/pub/epel/6/SRPMS
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-source-6&arch=$basearch
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
gpgcheck=1
```

5. Remiリポジトリの有効化オプションを永続的に使用できるようにする．

```bash
# CentOS7の場合
$ yum install -y yum-utils
# 永続的に有効化
$ yum-config-manager --enable remi-php74


# CentOS8の場合（dnf moduleコマンドを使用）
$ dnf module enable php:remi-7.4
```

6. remiリポジトリを指定して，php，php-mbstring，php-mcryptをインストールする．Remiリポジトリを経由してインストールしたソフトウェアは```/opt/remi/*```に配置される．

```bash
# CentOS7の場合
# 一時的に有効化できるオプションを用いて，明示的にremiを指定
$ yum install --enablerepo=remi,remi-php74 -y php php-mbstring php-mcrypt


# CentOS8の場合
# リポジトリの認識に失敗することがあるのでオプション無し
$ dnf install -y php php-mbstring php-mcrypt
```

7. 再インストールする時は，reinstallとすること．

```bash
# CentOS7の場合
# 一時的に有効化できるオプションを用いて，明示的にremiを指定
$ yum reinstall --enablerepo=remi,remi-php74 -y php php-mbstring php-mcrypt


# CentOS8の場合
# リポジトリの認識に失敗することがあるのでオプション無し
$ dnf reinstall -y php php-mbstring php-mcrypt
```

<br>

## 04. 言語バージョン管理ユーティリティ

### pyenv（Python）

#### ・which

```bash
# pythonのインストールディレクトリを確認
$ pyenv which python
/.pyenv/versions/3.8.0/bin/python
```
