---
title: 【IT技術の知見】Go＠ホワイトボックステスト
description: Go＠ホワイトボックステストの知見を記録しています。
---

# Go＠ホワイトボックステスト

## はじめに

本サイトにつきまして、以下をご認識のほど宜しくお願いいたします。

ℹ️ 参考：https://hiroki-it.github.io/tech-notebook-mkdocs/about.html

<br>

## 01. ホワイトボックステストのツール

### 整形

標準の```go fmt```コマンド

<br>

### 静的解析

標準の```go vet```コマンド

<br>

### ユニットテスト/機能テストツール

標準の```go fmt```コマンド

<br>

## 02. 標準のテストツール

### 標準のテストツールとは

```go```コマンドが提供するホワイトボックス機能のこと。

<br>

### 網羅率

網羅率はパッケージを単位として解析される。網羅については、以下のリンクを参考にせよ。

ℹ️ 参考：https://hiroki-it.github.io/tech-notebook-mkdocs/testing/testing_whitebox_php.html

<br>

## 03. 設計ポリシー

### パッケージ名

#### ▼ ホワイトボックステスト

テストファイルのパッケージ名が、同じディレクトリ配下にある実際の処理ファイルのパッケージ名と同じ場合、それはホワイトボックステストになる。

#### ▼ ブラックボックス風のホワイトテスト

テストファイルのパッケージ名が、同じディレクトリ配下にある実際の処理ファイルに『```_test```』を加えたパッケージ名の場合、それはブラックボックステスト風のホワイトテストになる。ちなみに、Goでは1つのディレクトリ内に1つのパッケージ名しか宣言できないが、ブラックボックステストのために『```_test```』を加えることは許されている。

ℹ️ 参考：https://medium.com/tech-at-wildlife-studios/testing-golang-code-our-approach-at-wildlife-6f41e489ff36

<br>

### インターフェースの導入

テストできない構造体はモックに差し替えられることなる。この時、あらかじめ実際の構造体をインターフェースの実装にしておく。テスト時に、モックもインターフェイスの実装とすれば、モックが実際の構造体と同じデータ型として認識されるようになる。これにより、モックに差し替えられるようになる。

<br>

### テーブル駆動テスト

ℹ️ 参考：https://github.com/golang/go/wiki/TableDrivenTests

<br>

### 回帰テスト

回帰テストを実現するため、過去のテスト結果をテストデータを保存しておき、今回のテスト結果が過去のものと一致するかを確認する。Goでは、このテストデータをファイルを『Golden File』という。Golden（金）は化学的に安定した物質であることに由来しており、『安定したプロダクト』とかけている。回帰テストについては、以下のリンクを参考にせよ。

<br>

### POSTデータの切り分け

POSTリクエストを受信するテストを行う時に、JSONデータをファイルに切り分けておく。これを```ReadFile```関数で読み出すようにする。

**＊実装例＊**

```go
package test

import (
	"io/ioutil"
)

/**
 * mainメソッドをテストします。
 */
func TestMain(t *testing.T) {
	// jsonファイルの読み出し
	data, err := ioutil.ReadFile("../testdata/data.json")

	// 以下にテストコードを実装していく
}
```

<br>
