﻿---
title: SCOM で使用する一般的なログ収集方法
date: 2022-11-25 14:11:00
tags:
  - SCOM
  - Operations Manager
  - Troubleshoot
  - トラブルシューティング
  - ログ収集
---

本記事は SCOM でトラブルシューティングを行う際、調査を行うために収集する一般的なログとその取得方法を解説いたします。

<!-- more -->
皆さまこんにちは。
System Center サポートチームの保科です。 

System Center Operations Manager (SCOM) をご利用中に様々な問題に対面された経験はございませんでしょうか。
この問題を対処するにあたり、弊社までお問合わをいただくことも多いかと存じます。
基本的にご申告いただきました問題を対処するにあたってログ等を含めた調査が必須となります。
本記事では、弊社が SCOM 観点のトラブルシューティングを行う際に一般的にお客様へ採取依頼を行うログとその採取方法について解説いたします。


## イベントログ
イベントログの調査は、SCOM のトラブルシューティングを行うにあたって最も一般的なログとなります。
基本的に SCOM で発生した問題の多くとその詳細はイベントログにも転記されます。
そのため、多くの問題はイベントログを調べることで原因の特定を行える可能性があります。
イベントログを採取いただく場合は、状況に応じて以下コンピューターから採取を行います。
- SCOM 管理サーバー:
トラブルシューティングの種類に依らず、必ず SCOM 管理サーバーのイベントログを採取します。
ご利用の環境で複数台の SCOM 管理サーバーをご利用の場合、それらすべての SCOM 管理サーバーからイベントログの採取を行います。
ゲートウェイサーバーを介した別ドメインコンピューターの監視で問題が発生されている場合、ゲートウェイサーバーからもイベントログを採取します。
- 事象が発生した監視対象コンピューター:
監視の問題が発生している場合、その問題が発生しているすべての監視対象コンピューターよりイベントログを採取します。
特定の監視対象コンピューターの問題ではなく、SCOM 全体的な問題によるトラブルシューティングを行う場合は、こちらのコンピューターからのログ採取は不要です。
- SCOM データベースが動作する SQL Server 実行サーバー:
データベースの問題が疑われる場合、このコンピューターからイベントログを採取します。
一般的には、SCOM の処理が遅い、データが欠落する、SCOM が機能しないといった場合にこれらのサーバーからのイベントログ採取が必要となります。
SCOM 管理サーバーの中に SCOM データベースが動作する SQL Server が存在する場合、こちらについては意識いただかなくても問題ございません。

イベントログの取得方法は以下手順で行います。
1. 対象コンピューターでイベント ビューアーを開きます。
確実に開く方法としては、[win] + [R] キーを押下し、"eventvwr.msc" とイベントビューアーの名前を直接指定し、実行する方法がございます。
2. 画面左のコンソールツリーより、保存するイベントログをクリックします。
SCOM 観点のイベントログは、以下 3 種類のイベントログを採取します。
   - アプリケーション (※1)
   - システム (※1)
   - Operations Manager (※2)

   (※1) は [Windows ログ] 配下に存在します。
   (※2) は [アプリケーションとサービス ログ] 配下に存在します。
   SCOM 管理サーバー / ゲートウェイサーバー以外のサーバーで、かつ SCOM エージェントがインストールされていない環境では (※2) のイベントログは存在しない場合がございます。
   この場合、このコンピューターからの (※2) のイベントログの採取は不要でございます。
3. 画面右ペインの [すべてのイベントを名前をつけて保存] をクリックします。
4. 保存するイベントログの任意の名称を指定します。
ファイルの種類としては、"イベント ファイル (*.evtx)" および "CSV (コンマ区切り) (*.csv)" の両方で取得します。

イベントログ取得手順は以上となります。
弊社までイベントログを送付いただく場合は、コンピューター毎にイベントログを 1 つのフォルダに纏め、それらのフォルダをさらに 1 つのフォルダに纏めて圧縮いただいてお寄せくださいませ。
採取対象コンピューターが複数台存在する場合、イベントログ自体のファイル名やそれを格納するフォルダにコンピューター名を付与いただく等、採取対象が分かりやすく識別できるように送付をいただけると幸いです。

## アラート情報
アラートの状態 (オープンの状態、解決済みの状態等) によらず、SCOM データベースに存在するアラート情報を出力いただけます。
こちらもイベントログ程ではありませんが、トラブルシューティングを行う際に有用な情報が確認できる場合が多い資料です。
アラート情報を取得する場合は、Operations Manager Shell を利用し、CSV 形式で一覧として取得します。
1. SCOM 管理サーバー、SCOM 管理者権限を持つアカウントでログインします。
2. 事前に、C ドライブの直下に “temp” という名前のフォルダを作成します。
3. [スタート] メニューより [Operations Manager Shell] を実行します。
4. 以下のコマンドを実行します。
```
Get-SCOMAlert | export-csv -path "c:/temp/Alert.csv" -encoding UTF8
```
5. 出力された "Alert.csv" を弊社までお寄せください。

## インストールログ
インストールログの調査は、SCOM のインストールやエージェントのプッシュインストール等インストールに関する操作が正常に行えない場合に調査する資料となります。
インストールに関する問題が発生しましたら、こちらのログを調査する必要があります。
ログは以下のフォルダに格納されております。
> C:\Users\<SCOM をインストールする際に使用したユーザー名>\AppData\Local\SCOM

## 管理パック
[こちらの記事](/blog/SCOM/SCOM_MPTransfer/) でも書かれております通り、SCOM の監視動作は基本的に管理パック内で定義されます。
例えば以下の状況が発生した場合においては、ご利用の管理パック側から調査を行う必要があります。
- 検出されると想定されていた内容が検出されない。
- 今まで出力されなかったアラートが出力されるようになった。
また、このアラートが解消と発生を繰り返している。
- 認識していなかったコマンドが監視対象サーバーで実行されているように見えるため、このコマンドが SCOM から実行されている可能性があるか調査したい。

管理パックは、以下手順でご利用の全管理パックを xml 形式での出力と、ご利用の管理パックを一覧化した html ファイルを出力いただけます。
1. 管理者の権限を持つユーザーで Operations Manager 管理サーバーにログオンします。
2. C: ドライブの直下に MPs フォルダーを作成します。
3. [スタート] 画面から [Operations Manager Shell] を選択します (※)。
4. [Operations Manager Shell] ウィンドウのプロンプトで以下のコマンドを実行します。
```
Get-SCOMManagementPack | Export-SCOMManagementPack -Path:C:\MPs
Get-SCOMManagementPack | ConvertTo-HTML > C:\MPs\MpList.html
```


4\. の 1 行目のコマンドを実行することで、ご利用の環境に導入されているすべての管理パックを xml 形式でエクスポートいただけます。
xml 形式でエクスポートされた管理パックには、その管理パックで定義されている内容がすべて記載されております。
基本的に SCOM で行われる監視内容は Powershell や VBS、Javascript 等で作成されたスクリプトを、対象となるコンピューターで実行するものとなります。
これらの実装を確認することで、管理パックに関するトラブルシューティングが可能になるというものです。

こちらの手順で、ご利用の管理パックが最新であるかを確認することもできます。
ご申告いただきました事象が、そのバージョンにおいて既知の事象であり、管理パックのアップデートによって解消される可能性もございます。
この場合、管理パックのアップデートによってご申告いただきました事象が解消される可能性が高いため、管理パックのアップデートをお願いいたします。
なお、このような既知の事象は、ほとんどの管理パックの場合はその管理パックを入手するサイトで入手いただける管理パックドキュメントに記載されております。
ドキュメントはほとんどの場合は英語での記述となりますが、ワークフローの一覧等管理パックに関する有用な内容が多く記載されております。

なお、一部お客様におかれましては、弊社が提供しております管理パック以外にも、サードパーティによって作成された管理パックを導入されている可能性がございます。
管理パックを調査した結果、このサードパーティ製の管理パックによってご申告いただきました事象が発生されている場合、弊社での詳細な調査はいたしかねております。
サードパーティ製の管理パックは、その管理パックを開発されたベンダーへ別途ご確認いただく必要がございます。
サードパーティ製の管理パックに関する調査をご所望でございましたら、その管理パックについて弊社で調査することが出来ないことをあらかじめご認識いただけます様お願いいたします。

## トレースログ
上記までに記載したログは、いずれも事象発生後に後追いで原因調査を行うために採取される資料となります。
これらの資料で大半の事象は原因を特定することが出来ますが、中には事象発生後に採取される資料に情報が一切記載されず、調査が困難な場合がございます。
この場合、事象発生時のログをリアルタイムで収集し、そのログから SCOM 内部で行われた処理を時系列で調査する必要があります。
この調査は SCOM 標準で備わるトレースログの採取によって可能となります。
以下手順にて SCOM トレースログの取得を開始します。
1. トレースログを取得するコンピューターに、管理者権限でログインします。
2. 管理者としてコマンドプロンプトを開きます。
3. cdコマンドを使用して、 トレースログを採取するコンピューターに応じて以下ディレクトリに移動します。
なお、下記のディレクトリは、いずれも既定フォルダにインストールを行った際に使用するディレクトリになります。
SCOM 管理サーバーや SCOM エージェントのインストール先を既定値から変更されている場合、適宜そのフォルダに移動をお願いします。
| トレースログ採取対象コンピューター | cd コマンドで移動するディレクトリの既定値 |
| --- | --- |
| SCOM 2016 管理サーバー | C:\Program Files\Microsoft System Center 2016\Operations Manager\Server\Tools |
| SCOM 2019 管理サーバー | C:\Program Files\Microsoft System Center\Operations Manager\Server\Tools| 
| SCOM 2022 管理サーバー | C:\Program Files\Microsoft System Center\Operations Manager\Server\Tools |
| SCOM エージェント | C:\Program Files\Microsoft Monitoring Agent\Agent\Tools |
4. 以下コマンドを実行し、すでに実行中のトレースログ採取を停止します。
   ```
   StopTracing.cmd
   ```
   SCOM トレースログは、"Microsoft Monitoring Agent" サービスが停止中の状態から起動状態に移行すると、自動的にエラーレベルのトレースログを収集開始します。
   "サービスが停止中の状態から起動状態に移行" する条件として、一例として以下が挙げられます。
   - サービス自体を手動で再起動する
   - 対象サーバーを再起動する
5. Windowsエクスプローラーを使用して、“C:\Windows\Logs\OpsMgrTrace” に移動します。
6. フォルダ “C:\Windows\Logs\OpsMgrTrace” のすべてのファイルを削除します。
7. 採取するトレースログのレベルに応じて以下コマンドを実行します。
   - 詳細レベルのトレースログ:
   ```
   StartTracing.cmd VER 
   ```
   - デバッグレベルのトレースログ:
   ```
   StartTracing.cmd DBG
   ```
   基本的に詳細レベルよりデバッグログのトレースログの方がより多くの情報が含まれます。
   そのため調査に際してはデバッグログの方がより有用な情報が採取される可能性が高いものの、それに比例してログ容量増加速度も上昇します。
   大まかな考えとしては、即座に事象再現が可能であればデバッグレベルのトレースログを、いつ事象再現が確認されるかが未定である場合は詳細レベルのトレースログを採取する方法を推奨いたします。
8. 事象を再現します。
再現が確認されるまでログを採取し、再現が確認されましたら 9. の手順へ進みます。
9. 以下コマンドを実行し、トレースログの採取を再度停止します。
```
StopTracing.cmd
```
10. 以下コマンドを実行し、採取されたトレースログをテキストファイル形式にフォーマットします。
```
FormatTracing.cmd R
```
11. 弊社まで資料を送付いただく際は、“C:\Windows\Logs\OpsMgrTrace”フォルダごとzip化し、弊社までお寄せください。
この際、複数のコンピューターでトレースログを採取いただけましたら、採取されたコンピューターが判別できるようにコンピューター名を zip ファイルに記載いただく等していただけますと幸いです。


(※1) 
“C:\Windows\Logs\OpsMgrTrace” フォルダ内には、収集された診断ログが “etl” 拡張子のファイルにて格納されております。
“.etl” の各ファイルは、100MB を上限としてログが保存されます。
ログが 100MB を超える場合、古いログから順番に新しいログへ上書きされます。
100 MB を超えるまでの時間は、環境に応じて変化いたしますので、一概のご案内が困難でございます。
弊社としては、問題の再現実施後に、ただちに 9. の手順でトレースの停止を実施いただきたく思います。
トレースログ採取容量は、トレースログ採取を開始する "StartTracing.cmd" の内容を直接編集することで増減が可能です。
具体的には、対象ファイル 99 行目に記載されている以下記述より、"-cir" パラメーターに続く数値が、トレースログ容量上限値を MB 単位で指定するものとなります。
> if not [%1]==[TracingGuidsConfigService] tracelogsm.exe -start %1 -flag %2 -level %3 -f %OpsMgrTracePath%\%1.etl -b 64 -ft 10 **-cir 100** -guid %4

(※2)
7. の手順で SCOM 診断ログを取得する際、特にパフォーマンスへの大きな影響が発生しないことを確認しております。

(※3)
トレースログを仕掛けたタイミングでサーバーの再起動等 SCOM エージェントサービスの停止が伴う操作を実施いただくと、トレースログの採取がリセットされ、OS 起動時に取得されるエラーレベルでのトレース採取が開始されます。
そのため、トレースログ採取中に SCOM エージェントサービスの停止が伴う操作を実施いただく場合、再度 1. から 8. までの手順を実施いただく必要がございます。

参考:
[診断トレースを使用する - Operations Manager | Microsoft Learn](https://docs.microsoft.com/ja-jp/system-center/scom/manage-overview-management-pack?view=sc-om-2022)
(SCOM 診断ログについて記載された弊社ドキュメントです。)