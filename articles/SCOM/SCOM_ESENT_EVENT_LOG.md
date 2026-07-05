---
title: SCOM で収集した ESENT イベント ログの説明が文字化けする事象について
date: 2026-06-22 09:00:00
tags:
  - SCOM
  - Trubleshooting
---

皆様こんにちは、System Center サポート チームです。
今回は、System Center Operations Manager (SCOM) で Windows Server 2022 または Windows Server 2025 から ESENT イベント ログを収集した際に、SCOM コンソール上でイベントの説明が文字化けして表示される事象についてご紹介します。

<!-- more -->
本事象については、System Center Operations Manager のリリース ノートにも既知の問題として記載されています。
・[Release notes for System Center Operations Manager](https://learn.microsoft.com/en-us/system-center/scom/release-notes-om?view=sc-om-2025#esent-event-log-descriptions-appear-garbled)

本記事では、リリース ノートの内容を補足する形で、事象の概要、発生条件、原因、および現時点での考慮事項について説明します。
## 事象の概要
SCOM でイベント ログ収集ルールを構成し、Windows Server 2022 または Windows Server 2025 上で記録された ESENT イベントを収集した場合、SCOM コンソールのイベント ビュー上で Description が文字化けすることがあります。

## 確認している発生条件
弊社で確認している範囲では、以下の OS から ESENT イベントを収集した場合に本事象が発生します。
* Windows Server 2022
* Windows Server 2025

また、事象は主に以下のような条件で確認されています。
* SCOM で Windows イベント ログを収集するルールを構成している
* 監視対象サーバーで ESENT ソースのイベントが記録される
* SCOM コンソールのイベント ビューで、収集された ESENT イベントの Description を確認する
* SCOM の表示言語が日本語になっている

なお、すべてのイベント ソースで同様に発生するものではなく、ESENT イベントで確認されている事象です。

## 原因
本事象は、Windows Server 2022 以降における ESENT イベントのメッセージ リソースの文字コードと、SCOM エージェント側のメッセージ文字列処理の組み合わせによって発生します。
Windows のイベント メッセージは、イベント ソースに対応するメッセージ テーブル リソースを参照して組み立てられます。このメッセージ テーブル リソースでは、MESSAGE_RESOURCE_ENTRY という構造体が使用され、Flags の値によって文字列の形式が示されます。従来、SCOM エージェントはこの Flags が 0x1、つまり UNICODE であるかどうかを判断し、UNICODE でない場合は ANSI の文字列として処理していました。しかし、Windows Server 2022 以降の ESENT イベントでは、メッセージ テーブル リソース内の MESSAGE_RESOURCE_ENTRY の Flags が 0x2 (UTF-8) を示す値に変更されていることを確認しています。
このため、SCOM エージェントが UTF-8 の文字列を ANSI として処理してしまい、日本語などのマルチバイト文字が正しく変換されず、SCOM コンソール上で文字化けして表示されます。

## 影響範囲
本事象による主な影響は、SCOM コンソール上で表示される ESENT イベントの説明が文字化けすることです。
イベントの発生自体、イベント ID、ソース、レベル、記録日時、コンピューター名などの基本的なイベント情報については、通常どおり収集されます。そのため、イベントの発生有無やイベント ID を条件とした監視自体が直ちに失敗する事象ではありません。
ただし、SCOM コンソール上でイベントの説明文を確認して内容を判断する運用を行っている場合、説明文の判読が難しくなる可能性があります。

## 対処および回避策について
残念ながら現時点でサポートされる対処法や回避策はございません。

## 参考情報
[Release notes for System Center Operations Manager](https://learn.microsoft.com/en-us/system-center/scom/release-notes-om?view=sc-om-2025#esent-event-log-descriptions-appear-garbled)

---

本記事の内容は以上となります。

Windows Server 2022 または Windows Server 2025 の ESENT イベントを SCOM で収集しており、SCOM コンソール上でイベント説明が文字化けして表示される場合は、本記事の内容をご確認ください。