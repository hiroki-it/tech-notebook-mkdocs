---
title: 【知見を記録するサイト】phpコマンドと設定
---

# phpコマンドと設定

## はじめに

本サイトにつきまして，以下をご認識のほど宜しくお願いいたします．

参考：https://hiroki-it.github.io/tech-notebook-mkdocs/about.html

<br>

## 01. コマンド

### -i

PHPの設定を表示する．

**＊例＊**

```bash
$ php -i
phpinfo()
PHP Version => 7.4

# 〜 中略 〜

```

出力量が多いため，```grep```を用いて，特定の項目のみを表示するようにすると良い．

```bash
# PHPのプロセスが使用可能なめもりの
$ php -i | grep memory_limit
memory_limit => 2048M => 2048M
```

<br>

### --ini

Configuration Fileの項目で，```php.ini```ファイルのあるディレクトリを表示する．

**＊例＊**

```bash
$ php --ini

Configuration File (php.ini) Path: /usr/local/etc/php # iniファイルのあるディレクトリ
Loaded Configuration File:         (none)
Scan for additional .ini files in: /usr/local/etc/php/conf.d
Additional .ini files parsed:      /usr/local/etc/php/conf.d/docker-php-ext-bcmath.ini,
/usr/local/etc/php/conf.d/docker-php-ext-pdo_mysql.ini,
/usr/local/etc/php/conf.d/docker-php-ext-sodium.ini

$ ls -la /usr/local/etc/php
total 164
drwxr-xr-x 1 root root  4096 Sep  1  2020 .
drwxr-xr-x 1 root root  4096 Sep  1  2020 ..
drwxr-xr-x 1 root root  4096 Sep 25 12:22 conf.d
-rw-r--r-- 1 root root 72278 Sep  1  2020 php.ini-development # 開発環境用iniファイル
-rw-r--r-- 1 root root 72582 Sep  1  2020 php.ini-production  # 本番環境用iniファイル
```

<br>

### -m

インストールされているモジュールを表示する．

**＊例＊**

```bash
$ php -m

[PHP Modules]
bcmath
Core
ctype

# ～ 中略 ～

xmlreader
xmlwriter
zlib

[Zend Modules]
```

なお，実際に読み込まれているかどうかは，```get_loaded_extensions```メソッドで確認できる．`

参考：https://stackoverflow.com/questions/478844/how-do-i-see-the-extensions-loaded-by-php

```bash
$ php -r 'print_r(get_loaded_extensions());'

Array
(
    [0] => Core
    [1] => date
    [2] => libxml
    
    # 〜 中略 〜
    
    [33] => bcmath
    [34] => pdo_mysql
    [35] => sodium
)
```

<br>

### --ri

拡張モジュールの設定値を表示する．

**＊例＊**

```bash
$ php --ri=<拡張モジュール名>
```

```bash
$ php --ri=Core

Core

PHP Version => 7.4.9

Directive => Local Value => Master Value
highlight.comment => <font style="color: #FF8000">#FF8000</font> => <font style="color: #FF8000">#FF8000</font>
highlight.default => <font style="color: #0000BB">#0000BB</font> => <font style="color: #0000BB">#0000BB</font>
highlight.html => <font style="color: #000000">#000000</font> => <font style="color: #000000">#000000</font>
highlight.keyword => <font style="color: #007700">#007700</font> => <font style="color: #007700">#007700</font>
highlight.string => <font style="color: #DD0000">#DD0000</font> => <font style="color: #DD0000">#DD0000</font>
display_errors => STDOUT => STDOUT
display_startup_errors => Off => Off
enable_dl => On => On

...

zend.detect_unicode => On => On
zend.signal_check => Off => Off
zend.exception_ignore_args => Off => Off
```

<br>

## 02. 設定ファイル（Docker PHP）

### ```php.ini```ファイル

#### ・```php.ini```ファイルとは

PHPの設定値を定義する．Docker PHPでは，```/usr/local/etc/php```ディレクトリに配置された```php.ini```ファイルや，追加ディレクト以下に配置された任意の```ini```ファイルに実装された設定値が，ユーザ定義のカスタム値として読み込まれる．また，それ以外の設定値はデフォルト値となる．

参考：https://www.php.net/manual/ja/configuration.file.php

```bash
$ php --ini

Configuration File (php.ini) Path: /usr/local/etc/php
Loaded Configuration File:         (none) # iniファイルがまだ配置されていない
Scan for additional .ini files in: /usr/local/etc/php/conf.d
Additional .ini files parsed:      /usr/local/etc/php/conf.d/docker-php-ext-bcmath.ini,
/usr/local/etc/php/conf.d/docker-php-ext-pdo_mysql.ini,
/usr/local/etc/php/conf.d/docker-php-ext-sodium.ini
```

Docker PHPでは，```/usr/local/etc/php```ディレクトリには```php.ini-development```ファイルと```php.ini-production```ファイルが最初から配置されている．これをコピーして設定値を変更し，読み込まれるようにファイル名を```php.ini```に変えて配置する（これ以外のファイル名でｊは読み込まれない）．あるいは，最小限の設定値のみを変更した```php.ini```ファイルを自身で作成し，同じく配置しても良い．

```bash
$ ls -la /usr/local/etc/php

drwxr-xr-x 1 root root  4096 Dec  2 13:39 .
drwxr-xr-x 1 root root  4096 Dec  2 13:39 ..
drwxr-xr-x 1 root root  4096 Dec 17 15:21 conf.d
-rw-r--r-- 1 root root 72382 Dec  2 13:39 php.ini-development
-rw-r--r-- 1 root root 72528 Dec  2 13:39 php.ini-production
# php.iniファイルをここに配置する
```

#### ・開発環境用```php.ini```ファイル

あらかじめ用意されている```php.ini-development```ファイルを参考に設定する．元の値をコメントアウトで示す．

参考：https://qiita.com/ucan-lab/items/0d74378e1b9ba81699a9

```bash
# 開発環境では，スタックトレースを表示
zend.exception_ignore_args = off # on

expose_php = on

max_execution_time = 30

max_input_vars = 1000

upload_max_filesize = 64M # 2M

post_max_size = 128M # 8M

memory_limit = 256M # 128M

# 開発環境では，全てのログレベルを出力
error_reporting = E_ALL # NULL

display_errors = on

display_startup_errors = on

# 開発環境では，ログをerror_log値の場所に出力
log_errors = on # 0(off)

# 開発環境では，エラーログをファイルに出力
error_log = /var/log/php/php-error.log # NULL

default_charset = UTF-8

[Date]
date.timezone = Asia/Tokyo # GMT

[mysqlnd]
# 開発環境では，メモリのメトリクスを収集
mysqlnd.collect_memory_statistics = on # off

[Assertion]
zend.assertions = 1

[mbstring]
mbstring.language = Japanese
```

#### ・本番環境用```php.ini```ファイル

あらかじめ用意されている```php.ini-production```ファイルを参考に設定する．元の値をコメントアウトで示す．

参考：https://qiita.com/ucan-lab/items/0d74378e1b9ba81699a9

```bash
zend.exception_ignore_args = on

# 本番環境では，X-Powered-ByヘッダーのPHPバージョンを非表示
expose_php = off # on

max_execution_time = 30

max_input_vars = 1000

upload_max_filesize = 64M

post_max_size = 128M

memory_limit = 256M

# 本番環境では，特定のログレベルを出力
error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT # NULL

# 本番環境では，エラーを画面に非表示
display_errors = off # on

display_startup_errors = off

# 本番環境では，エラーログをerror_log値の場所に出力
log_errors = on # 0(off)

 # 本番環境では，エラーログを標準エラー出力に出力
error_log = /dev/stderr # NULL

default_charset = UTF-8

[Date]
date.timezone = Asia/Tokyo

[mysqlnd]
mysqlnd.collect_memory_statistics = off

[Assertion]
zend.assertions = -1

[mbstring]
mbstring.language = Japanese

# 本番環境では，Opcache機能を有効化
[opcache]
opcache.enable = 1
opcache.memory_consumption = 128
opcache.interned_strings_buffer = 8
opcache.max_accelerated_files = 4000
opcache.validate_timestamps = 0
opcache.huge_code_pages = 0
opcache.preload = /var/www/preload.php
opcache.preload_user = www-data
```

<br>

## 03. 役立つ機能

### OPcache

#### ・OPcacheとは

通常，PHPのソースコードは実行の度にバイナリ形式のコードに変換される．バイナリ形式のコードのキャッシュを作成しておき，ソースコードが変更された時だけ変換するようにする．これにより，PHPのソースコードの実行が高速化される．

参考：https://weblabo.oscasierra.net/php-opcache/