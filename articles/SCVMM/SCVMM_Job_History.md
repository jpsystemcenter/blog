---
title: SCVMM ジョブ保存設定変更方法
date: 2022-07-07 12:00:00
tags:
  - SCVMM
  - HowTo
  - SCVMM Server
  - Job
disableDisclaimer: false
---

<!-- more -->
皆さんこんにちは、System Center サポートチームの 保科と申します。


今回は、System Center Virtual Machine Manager (SCVMM) で保存されるジョブ履歴の保存設定についてご紹介いたします。

### 本ページが対象とする SCVMM バージョン
SCVMM 2016
SCVMM 2019
SCVMM 2022

### 概要
SCVMM で実行される処理の多くは、ジョブという形で実行されます。
このジョブが実行されると、実行可否に関わらず、履歴としてすべて SCVMM データベースに存在する [dbo].[tbl_TR_TaskTrail] というテーブルに保存されます。
ジョブ履歴ですが、バッチ処理を組まれる等して大量のジョブが短期間で実行される環境においては、このテーブルが肥大化する要因となります。
テーブルが肥大化することで、SCVMM 観点においては下記の影響が発生します。
- 単純に [dbo].[tbl_TR_TaskTrail] テーブルサイズが肥大化し、データベースファイルサイズの肥大化
- SCVMM へのパフォーマンスへの影響

意外に思われるかもしれませんが、ジョブ履歴が大量に保存されている環境においては、SCVMM へのパフォーマンス影響が発生する可能性がございます。
特段負荷が大きい処理を連続して実施されていないにも関わらず、SCVMM ジョブの実行速度や SCVMM 自体が以前より遅くなったといった状況では、ジョブ履歴が大量に保存されている可能性がございます。

### 対処手順
このジョブ履歴ですが、レジストリ値によってジョブ履歴保持日数および削除処理が行われる周期を設定いただけます。
そのため、"概要" 項で説明しました影響が発生されている場合は、そのレジストリ値設定を変更いただくのが切り分けとして有用となります。

追加いただくレジストリは、以下の 2 種類が該当いたします。

> レジストリキー:
> HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft System Center Virtual Machine Manager Server\Settings\Sql
>  
> レジストリの種類:
> DWORD (32 ビット) 値
>  
> レジストリの名称: 
> TaskGC
> 
> レジストリ値: 
> ジョブの保持日数を整数値で指定します。
> 例えば、本レジストリ値が 7 に設定された場合、ジョブ履歴削除処理が行われた日付時刻から 7 日以内のジョブは保持され、それ以外はすべて削除対象となります。
> レジストリが未作成の状態である場合、本レジストリ値は 90 に設定されたものとして動作が行われます。
> こちらのレジストリ設定値を、ご利用の環境でどの程度ジョブ履歴を重要視されるかに応じて設定します。
> また、本レジストリ値の設定値に関わらず、一度のジョブ履歴削除処理で削除されるジョブ数は最大で 25000 件となります。
> そのため、各処理間で 25000 件以上のジョブ履歴が保存される場合、ジョブ履歴は傾向として増加される状況となります。
> ご利用の環境に合わせて、もう一つのレジストリである "DBGCUpdateInterval" のレジストリ値を構成します。

> レジストリキー:
> HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft System Center Virtual Machine Manager Server\Settings
>  
> レジストリの種類:
> DWORD (32 ビット) 値
>  
> キーの名称 : 
> DBGCUpdateInterval
>  
> レジストリ値: 
> ジョブ削除ジョブが実行される周期を秒単位の整数値で指定します。 
> レジストリが未作成の状態である場合、本レジストリ値は 72000 (= 20 時間) に設定されたものとして動作が行われます。

レジストリの作成は、下記手順で行えます。
1. レジストリを追加する SCVMM サーバーに管理者権限でログインします。
2. [Win] + [R] キーを押下し、"regedit" と入力して実行します。
3. 画面左ペインより、各レジストリの追加先となるレジストリキーにアクセスします。
4.  レジストリが表示された画面上で右クリックし、[新規] -> [DWORD (32 ビット) 値] をクリックします。
5. 追加されたレジストリに対して、”キーの名称” で指定された文字列を入力します。
6. 追加されたレジストリをダブルクリックし、レジストリの編集を行います。
7. ウィンドウ右側に表示された [10 進数] を選択し、レジストリで指定いただく値を入力します。
8. レジストリ追加完了後、SCVMM サーバーに関するサービスである “System Center Virtual Machine Manager” サービスを再起動します。

### 厳密な処理内容
ジョブ履歴削除処理は、SCVMM データベースに作成された "dbo.prc_TR_GarbageCollect" ストアド プロシージャが呼び出されて実行されます。
最大25000 行のデータ削除処理は、ストアド プロシージャ内に下記の通り "SELECT TOP 25000..." と、実装が明示的に行われております。
> (途中略)
> \-\- do at most 25,000 every time to handle extreme backlog cases
> \-\- get the list of IDs to delete
> SELECT TOP 25000 TaskID, ResultObjectID
> INTO #TasksToDelete
> FROM dbo.tbl_TR_TaskTrail
> WITH (ROWLOCK, UPDLOCK, HOLDLOCK)
> WHERE EndDateTime IS NOT NULL AND EndDateTime \< @CutoffDateTime
> (途中略)
>
> (こちらの実装は SCVMM 2016、2019 および 2022 で同一です。)


### ご参考
既に弊社にて公開がされていないサイトとなりますので、こちらのブログにて併せてそちらのサイトの内容をご紹介いたします。
その公開終了されたサイトでは、SCVMM のジョブについて要約すると以下の内容が書かれておりました。
- SCVMM の仕様として、目安ではあるものの 10000 の同時並行実施ジョブが処理可能であること。
- 10000 以上のジョブが同時並行で実施されている場合、SCVMM サーバーのパフォーマンスに多大な影響を及ぼす可能性があること。
- 大量のジョブが並行実施される動作とは別に、SCVMM ジョブ履歴が大量に保存されている場合、それによっても SCVMM サーバーのパフォーマンスに多大な影響を及ぼす可能性があること。

公開終了とされる前に記載されております内容について、原文のままご紹介いたします。
- SCVMM で同時並行で実行可能なジョブ数に関する記述:
> The maximum number of running jobs that an SCVMM system should be able to handle is around 10,000 and hitting this many running jobs at one time would be an exceptionally rare occurrence. 
> Should it occur, however, it can have a severe impact on the performance of the SCVMM engine.

- SCVMM ジョブ履歴が大量に保存されていることで発生しうる影響に関する記述:
> The number of retained Jobs count refers to the number of running tasks tracked in the TaskTrail database table in SQL. 
> The size of this database can have as much of an impact on performance as the actual running jobs. 
> If the number of running jobs nowhere reaches 10,000 then it may be that the tombstone limit on entries tracked in the TaskTrail database is too conservative

### アップデート履歴
- 2023/11/13 更新:
一度のジョブ履歴削除処理が行われる際に削除されるデータの件を 50000 件から 25000 件に変更しました。
従前の記述である 50000 件は、SCVMM 2008 における仕様でございました。
また、データ削除処理の詳細も記載いたしました。