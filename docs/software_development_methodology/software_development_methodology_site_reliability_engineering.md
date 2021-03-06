---
title: 【IT技術の知見】SREing：Site Reliability Engineering
description: SREing：Site Reliability Engineeringの知見を記録しています。
---

# SREing：Site Reliability Engineering

## はじめに

本サイトにつきまして、以下をご認識のほど宜しくお願いいたします。

ℹ️ 参考：https://hiroki-it.github.io/tech-notebook-mkdocs/about.html

<br>

## 01. SREingの用語集

### SREingとは

#### ▼ 引用：[サイトリライアビリティエンジニアリング](https://www.amazon.co.jp/dp/4873117917)

サイトの信頼性向上を目的としたエンジニアリング手法のこと。

<br>

### サイトの『信頼性』とは

#### ▼ 引用：[サイトリライアビリティエンジニアリング](https://www.amazon.co.jp/dp/4873117917)

信頼性とは、『システムが求められる機能を、定められた条件下で、定められた期間にわたり、障害を起こすことなく実行する確率』のこと。

#### ▼ 引用：[SRE実践の手引](https://eh-career.com/engineerhub/entry/2019/12/05/103000)

SREerの具体的な行動を明確にするためには、サイトの信頼性を表す指標（SLI）と、指標の具体的な目標値（SLO）を定義する必要がある。エラーバジェットの残量を考慮に入れ、リスクを取りながら、サイトの信頼性の維持に努める。

#### ▼ 引用：[SREの基本と組織への導入](https://dev.classmethod.jp/articles/202105-report-gcd21-d3-infra-01/)

ソフトウェアの最も重要な機能は信頼性であり、信頼性の程度はサイトのユーザーによって決められるべきである。ユーザーは、SLIを信頼性の指標とし、SLOに至った場合に信頼できるサイトと見なす。

#### ▼ 引用：[組織の信頼性のマインドセット](https://cloud.google.com/blog/ja/products/devops-sre/the-five-phases-of-organizational-reliability)

プロダクトの信頼性は、そのシステムのアーキテクチャ/プロセス/文化/開発組織のマインドセットによって決まる。

<br>

### SREingが有用な業態

#### ▼ 引用：[サイトリライアビリティエンジニアリング](https://www.amazon.co.jp/dp/4873117917)

SREingが信頼性を向上させるソフトウェアは、サイトやそれに類似するサービスである。原子力発電所、航空機、医療機器、その他安全性が極めて重要なソフトウェアなどの信頼性については考えない。

#### ▼ 引用：[SRE実践の手引](https://eh-career.com/engineerhub/entry/2019/12/05/103000)

SREingには、継続的な改善によって、システムの信頼性を向上させようとする思想がある。そのため、アジャイル開発を採用できないシステム（例：人命にかかわるようなシステム）を扱っている業態や、サイトを扱っていても納品後に改善作業がないような業態（例：受託会社）では、有効に機能しない。

<br>

### SREingの土台となる文化

#### ▼ 引用：[SREの探求](https://www.amazon.co.jp/dp/4873119618)

チームが成功するか否かの最重要な指標は、在職期間の長さ、年功序列、給与水準の程度ではなく、心理的安全性の程度である。心理的安全性は、一般的に以下の時に低くなる。もちろん、個人で『安全』と感じる状態は異なるため、アンケートしても良い。

- もし失敗したら、チームは自分を非難するだろう。

- チームは独自の文化を持っており、新しい文化を持つメンバーが溶け込みにくい。

- チームは、困っているものに手を差し伸べるのが遅い。

- チームの目的を達成する上で、自分のスキルを発揮するよりも優先すべきことがある。

- チームのセンシティブな問題について正直に話し合うことは気まずい。

一方で技術には、より正しい理解やベタープラクティスがある。他の人の提案がこれと異なっていた時に、その人に心理的安全性を下げずに、正しい情報やベタープラクティスに軌道修正する方法が難しい。エンジニアに関係なく、ビジネススキルみたいな話になる。そこで、心理的安全性だけを最適化するわけではなく、チームの生産性などの他の要素も考慮に入れながら、バランスの良い最適化を図るようにする。

#### ▼ 引用：[SRE実践の手引](https://eh-career.com/engineerhub/entry/2019/12/05/103000)

改善のきっかけは失敗であることが前提にあるため、失敗が許されない文化がある会社組織でも機能しない。

<br>

### SREingの組み込み方

SREerチームとアプリケーションチームの間で、どのようにコミュニケーションを行なっていくかは、SREerの組み込み方によって異なる。

ℹ️ 参考：https://x-tech5.co.jp/2022/02/21/204/

<br>

## 02. その他情報収集

### 企業

#### ▼ Wantedly

SREチームのValueから、SREerに必要な技術がわかる。

ℹ️ 参考：https://gist.github.com/south37/85d97e02d7816a31053971d63c164880

#### ▼ 3-shake、Topotal

提供しているサービスから、SREerに必要な技術がわかる。

ℹ️ 参考：

- https://sreake.com/
- https://topotal.com/services/sre-as-a-service

<br>

### ニュース

#### ▼ SREウィークリー

ℹ️ 参考：https://sreweekly.com/about-sre-weekly-2/

