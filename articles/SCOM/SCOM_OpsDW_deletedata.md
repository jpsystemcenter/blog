﻿---
title: SCOM OperationsManagerDW データベースの容量削減手順
date: 2025-03-03 18:00:00
tags:
  - SCOM
  - Trouble
  - Database
  - OperationsManagerDW
disableDisclaimer: false
---

皆さんこんにちは、System Center サポートチームの 保科と申します。


今回は、System Center Operations Manager (SCOM) のデータウェアハウスデータベースのデータ削除手順についてご紹介いたします。

<!-- more -->
### 本ページが対象とする SCOM バージョン
SCOM 2016
SCOM 2019
SCOM 2022

### 概要
SCOM を最小限の構成でインストールすると、以下 2 種類のデータベースが作成されます。
- OperationsManager:
SCOM の監視構成やインポートされた管理パック、監視対象の情報、監視データやアラート情報等 SCOM に関するあらゆる情報が保存されるデータベースです。
このデータベースが動作不備になったり、容量がいっぱいになると、SCOM 自体を使用することが出来なくなります。
- OperationsManagerDW
SCOM のレポート機能で使用するための監視データや監視対象の情報等が保存されるデータベースです。
保存されるデータは、SCOM の監視動作に準ずるものとなります。
例えば、パフォーマンスカウンターの収集を実施されている場合、まずは "OperationsManager" データベースにその監視内容が保存されます。
その後、その監視内容が "OperationsManagerDW" 側でも保存され、更にレポート用のデータとしてその保存されたデータが加工されます。本データベースに不備が発生すると、レポートに関するデータが取得できない、つまりレポートを閲覧することが出来なくなります。
※ 補足
本データベースの主要用途はレポート機能ですので、本データベースに接続できない場合でも SCOM コンソールにログインして利用する各種機能 (パフォーマンス ビューの参照など) は一見、利用できているように見えます。それらの機能は主に "OperationsManager" データベースを利用するためです。しかしながら、SCOM では "OperationsManager", "OperationsManagerDW" の両方のデータベースが必須です。レポート機能を利用されない場合でも "OperationsManagerDW" は必要なデータベースです。

詳細は割愛しますが、"OperationsManagerDW" では監視データの全データ、時間単位で纏められたデータ、日単位でまとめられたデータが保存されます。
収集されるデータによってはデータ数が膨大となることがあり、これが要因でデータベースの容量が非常に大きくなる場合がございます。
上記に記載した "OperationsManagerDW" に対するデータ保存の仕組みから、一般的に SCOM の監視内容が多いほどデータベースが肥大化しやすい傾向となります。
本記事では、肥大化した "OperationsManagerDW" データベースのデータ削除方法と、具体的に削除するデータの選定方法を説明いたします。

### 注意事項
以降のご説明では、"OperationsManagerDW" データベースの名称を "OpsDW" と略称いたします。
SCOM インストール時に "OperationsManagerDW" データベースの名称を変更されている場合、適宜そのデータベース名に "OpsDW" を読み替えてください。


### 事前準備
クリーンアップの設定を実施いただく前に、まずどのデータセットが OpsDW のデータの多くを占めるか確認する必要がございます。
こちらは、以下の手順で OpsDW のテーブル毎のデータ使用量を確認することで確認いただけます。
1. OpsDW データベースを運用する SQL Server が存在するサーバーに管理者権限でログインします。
2. SQL Server Management Studio (SSMS) を開きます。
3. OpsDW の管理者権限を持つアカウントで、OpsDW が存在するサーバーにログインします。
4. OpsDW を画面左ペインより [<サーバー名>] -> [データベース] -> [OperationsManagerDW] を右クリックし、[レポート] -> [標準レポート] -> [上位のデーブルによるディスク使用量] をクリックします。
(OpsDW のデータベース名が変更されている場合、適宜そのデータベースを [<サーバー名>] -> [データベース] より選択します。)

こちらの手順で、現在 OpsDW にはどのようなテーブルが存在し、各テーブルでどの程度データを使用しているか確認いただけます。
これにより、表示されたテーブルの内、どのデータセットが OpsDW のデータの多くを占めているか確認します。
例えば、"perf.perfhourly_<テーブル毎の固有文字列>" が、固有文字列の内容こそ異なるものの "perf.perfhourly_" で始まるテーブルが大量に存在する場合を想定します。
この場合、"perf.perfhourly"、つまり下記の弊社サイト内に記載されているテーブルにおける "データセット" が "パフォーマンス" かつ集計の種類が "1 時間ごと" のデータが大量に存在することが確認いただけます。
[レポート データ ウェアハウス データベースのクリーンアップ設定を構成する方法 | Microsoft Learn](https://learn.microsoft.com/ja-jp/system-center/scom/manage-omdwdb-grooming-settings?view=sc-om-2022)

このように、適宜お客様環境における OpsDW で [上位のデーブルによるディスク使用量] を開き、どのテーブルが消費容量の多くを占めるか確認いただくことで、クリーンアップ設定実施方針が定義いただけます。

### 対処手順
[事前準備] に記載しました方法でどのデータが OpsDW において多くのデータを消費しているか確認いただけましたら、次の手順としてはこのクリーンアップの設定を行います。
クリーンアップの設定手順は、下記の手順にて実施いただけます。
1. OpsDW データベースを運用する SQL Server が存在するサーバーに管理者権限でログインします。
2. SQL Server Management Studio (SSMS) を開きます。
3. OpsDW の管理者権限を持つアカウントで、OpsDW が存在するサーバーにログインします。
4. 以下弊社公開情報の "レポート データ ウェアハウスのクリーンアップ設定を変更するには" 項に記載された 4. ~ 6. の手順に沿って、削除を実施するデータの GUID を確認します。
[レポート データ ウェアハウス データベースのクリーンアップ設定を構成する方法 | Microsoft Learn](https://learn.microsoft.com/ja-jp/system-center/scom/manage-omdwdb-grooming-settings?view=sc-om-2022)
本公開情報の 4. の手順は機械翻訳されて分かりづらいですが、"dbo.Dataset" テーブルを開くという手順が英語版の公開情報では示されています。
5. 画面左ペインより [<サーバー名>] -> [データベース] -> [OperationsManagerDW] -> [テーブル] -> [dbo.StandardDatasetAggregation] を右クリックし、[上位 200 行の編集] をクリックします。
(OpsDW のデータベース名が変更されている場合、適宜そのデータベースを [<サーバー名>] -> [データベース] より選択します。)
dbo.StandardDatasetAggregation のテーブルを編集開始します。
6. 4\. で確認された GUID を持つ行の“MaxDataAgeDays” カラムの各値を、お客様が保持されたいデータの日数で指定します。
保持日数変更対象となるデータセットおよび集計の種類の確認方法は、下記の弊社公開情報の内容に基づいて確認します。
[レポート データ ウェアハウス データベースのクリーンアップ設定を構成する方法 | Microsoft Learn](https://learn.microsoft.com/ja-jp/system-center/scom/manage-omdwdb-grooming-settings?view=sc-om-2022)
なお、お客様環境によってはディスクの空き容量がひっ迫している状況も想定されると存じます。
この場合、トランザクションログ用の領域も十分に確保することが出来ない可能性もございます。
これは、クリーンアップ処理が実行される際、処理のコミットが完了するまで、データがトランザクションログに保存されるためです。
(データベースの復旧モデルを、SQL Server Always On 可用性グループをご利用いただく等して "完全" で構成されている場合、トランザクションログはバックアップされない限り保存され続けます。)
こういった場合では、“MaxDataAgeDays” の値は目的の値に達成されるまで段階的に減らされることを推奨いたします。
例えば、最初の段階では 2 ずつ減らし、ディスクの空き容量が確保いただける状況となった場合に減らす数値を増加する、といった流れでデータ削除を実施ください。
7. 6\. で編集された行の “GroomingIntervalMinutes” に、非常に小さな値を指定します。まずは 5 で設定されることを推奨いたします。
これにより、その行で指定されているテーブルのデータを、指定した分間隔で、”MaxDataAgeDays” で指定した日数以上経過しているデータを削除します。
例えば、“GroomingIntervalMinutes” の値を 5、”MaxDataAgeDays” の値を 100 と設定すると、５分間隔で、100 日以上経過した対象テーブル内データを削除する処理を行います。
なお、“GroomingIntervalMinutes” の値が小さいほど、ログデータ削除の処理が行われる回数も増えるので、実運用において小さな値を指定し続けることは推奨いたしません。
なので、“GroomingIntervalMinutes” の値を変更する前の値については、あらかじめメモを取得の上、一通りクリーンアップが完了したタイミングで値を元に戻します。
本設定値の変更は、例えばディスクの空き容量を早急に確保する必要があるといったご要望が存在する場合、空き容量が確保できるまでクリーンアップ実行周期を短くします。
即座に空き容量を確保いただく必要がない場合、本手順の設定は省略いただいても問題ございません。
8. 必要に応じて、6\. で編集された行の ”MaxRowsToGroom” の値を変更します。
一度のクリーンアップによって対象のテーブルより削除されるデータ行数は、”MaxRowsToGroom” にて指定された行数となります。
例えば、State データセットのテーブルは、既定で ”MaxRowsToGroom” の値が 50,000 となっております。
つまり、この設定状況では、一度のクリーンアップで削除されるデータ行数は 50,000 行までとなります。
クリーンアップ量に対して入力されるデータ数が多い場合、データ数は減ることなく増加することが想定されます。
こういった状況が想定されますため、必要に応じて、”MaxRowsToGroom” の値を変更します。
事例より、”MaxRowsToGroom” の値を 1,000,000 と設定した場合に大きなトランザクションログの容量を消費しないことを確認しております。
こちらは “GroomingIntervalMinutes” とは異なり、一通りクリーンアップが完了した後も設定値を保持いただいても問題ございません。
なお、こちらの値についても、ディスク空き容量がひっ迫している場合、段階的に値を増加いただく等の操作をご検討ください。
9. 設定した GroomingIntervalMinutes の時間が経過後、OpsDW の消費容量が小さくなったか確認します。
容量が小さくなっていることを確認できましたら、OpsDW を圧縮し、ファイルサイズの縮小を行います。
データベースの圧縮手順は、下記の弊社 SQL Server の公開情報を参照ください。
[データベースの圧縮 - SQL Server | Microsoft Learn](https://learn.microsoft.com/ja-jp/sql/relational-databases/databases/shrink-a-database?view=sql-server-ver16)
データベースの圧縮が不要の場合、こちらの手順の実施も不要です。
10. 以後、各テーブルのデータ最大保持日数が目的の値となるまで、5. ~ 8. の手順を繰り返します。
なお、ディスクの空き容量が増加するに従い、”MaxDataAgeDays” にて減らされる値を大きくされたり、
“MaxRowsToGroom” の値を大きくされても問題ございません。

### 補足
SQL Server の復旧モデルが "完全" の場合、トランザクションログの肥大化が懸念されます。
復旧モデルは、SQL Server Always On を構成されていない場合、既定で “単純” となり、"単純" で運用されることが推奨されます。
なお、SQL Server Always On を構成されている場合、復旧モデルは "完全" で固定となり、それ以外の復旧モデルに変更いただくことは出来ません。。
復旧モデルは、下記の手順で確認を行います。
1. SQL Server Management Studio (SSMS) を開きます。
2. 復旧モデルを確認するデータべースが存在するサーバーに、管理者権限でログインします。
3. 対象のデータベースを右クリックし、[プロパティ] をクリックします。
ウィンドウ左の [オプション] をクリックし、 [復旧モデル] の内容を確認します。

### おわりに
OpsDW の不要なデータ削除手順は以上となります。
上記の通り、OpsDW は、主に SCOM レポートで使用するデータを保存するために使用されるデータベースとなります。
お客様が使用されているレポートの要件に応じて、各データの保持日数およびその削除量を調整いただければと思います。
