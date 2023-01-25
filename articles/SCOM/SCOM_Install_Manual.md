---
title: SCOM 管理サーバー構築手順 ～ 全画面ショット付き
date: 2023-01-25 10:00:00
tags:
  - SCOM
  - Operations Manager
  - インストール
---

皆様こんにちは、System Center サポートチームの 石原 です。 

System Center Operations Manager（以後 SCOM） は 1,000 台を超えるような大規模な IT インフラの監視をサポートする製品であり、IT インフラ環境の構成、規模、運用要件に応じて導入が必要な SCOM コンポーネントとその台数が変わりますが、検証等の目的で簡易的に環境を最短で構築されたいケースもあります。
そういったケースに備えて、本記事では、1 台の Windows Server に SCOM の主コンポーネントの SCOM 管理サーバーと SCOM コンソールをインストールして、SCOM 管理コンソールにサインインできるまでの手順を画面ショット付きで紹介します。
<!-- more -->

# SCOM 導入
# SCOM 導入の流れ
SCOM 導入の大まかな流れは下図の通りです。
最初に各種コンポーネントの導入を行い SCOM 環境を構築します。
SCOM 管理サーバーや SCOM コンソールなどの必須コンポーネントの導入の後、IT インフラ環境や監視要件に応じてオプション コンポーネントを導入します。
SCOM 環境の構築後、監視要件に合致した管理パックのインポートと監視対象機器の登録を行うことで監視が開始されます。
以降、運用フェーズにおいて監視設定等のチューニングを行います。
![](001.png)

※ 構築フェーズの計画ガイド、運用フェーズのタスクについては、以下のマニュアル サイトをご参照ください。
　[Operations Manager 計画ガイド](https://learn.microsoft.com/ja-jp/system-center/scom/plan-overview?view=sc-om-2022)
　[Operations Manager のタスクに関するクイック リファレンス](https://learn.microsoft.com/ja-jp/system-center/scom/manage-quick-reference?view=sc-om-2022)

本記事では、1 台の Windows Server にデータベース、SCOM 2022 管理サーバー、SCOM コンソールをインストールする手順 (上の図の赤枠で囲った部分の作業) を画面ショット付きで紹介します。
<span style="color: red; ">**ドメインに参加した Windows Server 2022  (メモリ 16 GB 以上)  を予め 1 台ご準備ください。**</span>

# データーベース (Microsoft SQL Server 2019) の導入
SCOM では、SCOM のシステム情報、監視設定情報、アラート情報、収集したパフォーマンスデータなどの保管にデーターベースを用います。
SCOM 管理サーバーをインストールする前に最初にデーターベースの準備が必要です。
使用するデーターベースは、Microsoft SQL Server 2019 ( ※累積的な更新プログラム 8 (CU8) 以降 ) をサポートします。
　**ご参考：**[SQL Server の要件](https://learn.microsoft.com/ja-jp/system-center/scom/plan-sqlserver-design?view=sc-om-2022#sql-server-requirements)


SCOM 用の SQL Server のインストール手順は以下の通りです。

## 1. SQL Server 2019 のインストール モジュールを準備
*① SQL Server 2019 のインストーラー*と *② CU8 以降の累積的な更新プログラム*をダウンロードして、SCOM をインストールする Windows Server 内にコピーします。
![](004.png)

*① SQL Server 2019 のインストーラー (評価版)* は [SQL Server のダウンロード](https://www.microsoft.com/ja-jp/sql-server/sql-server-downloads) よりダウンロードできます。
![](002.png)

ダウンロードした EXE ファイル (SQL2019-SSEI-Eval.exe) を管理者で実行すると以下の画面が表示されます。
メディアのダウンロードをクリックすることで、ISO ファイル形式のインストーラーをダウンロードできます。
![](003.png)

*② SQL Server 2019 の累積的な更新プログラム*は [Download SQL Server® 2019 for Microsoft® Windows の最新の累積的な更新プログラム](https://www.microsoft.com/ja-JP/download/details.aspx?id=100809) よりダウンロードできます。
本記事では、掲載日時点で最新の CU 18 を使用します。 

## 2. SQL Server 2019 のインストール
インストールファイルの準備ができましたら、実際にインストールを実行します。
1. SCOM をインストールする Windows Server にドメイン管理者でサインインします。
2. SQL Server 2019 のインストーラーを選択して、右クリック メニューでマウントします。
![](005.png)

3. マウント先のドライブで setup.exe を選択して、右クリック メニューの [管理者として実行] をクリックします。
![](006.png)

4. インストール画面を表示して、[SQL Server の新規スタンドアロン インストールを実行するか、既存のインストールに機能を追加] をクリックします。
![](007.png)

5. 本記事では Evaluation のまま [次へ] ボタンをクリックして進みます。
![](008.png)

6. ライセンス条項を確認して、 [次へ] ボタンをクリックして進みます。
![](009.png)

7. 本記事では [Microsoft Update を使用して更新プログラムを確認する] は未チェックのまま [次へ] ボタンをクリックして進みます。
![](010.png)

8. 事前検証でエラーが無いことを確認します。
[Windows ファイアウォール] の警告が出ていますが、本記事では SCOM 管理サーバーを同一サーバーにインストールするため SQL Server へのリモート接続は必須ではないことから、このまま [次へ] ボタンをクリックして進みます。 
![](011.png)

9. SCOM に必須の [データーベース エンジン サービス] と [検索のためのフルテキスト抽出とセマンティック抽出] を選択して  [次へ] ボタンをクリックして進みます。
![](012.png)

10. 本記事では [既定のインスタンス] のまま [次へ] ボタンをクリックして進みます。
![](013.png)

11. [サーバーの構成] の [サービス アカウント] タブ画面で [SQL Server エージェント] と [SQL Server Browser] のスタートアップの種類を [自動] にします。
また、[SQL Server データベース エンジン サービスにボリューム メンテナンス タスクを実行する特権を付与する] にチェックします。
![](014.png)

12. [サーバーの構成] の [照合順序] タブ画面で[サポートされた日本語の照合順序](https://learn.microsoft.com/ja-jp/system-center/scom/plan-sqlserver-design?view=sc-om-2022#windows-collation)  ( Japanese_CI_AS、もしくは Japanese_XJIS_100_CI_AS ) が指定されていることを確認して  [次へ] ボタンをクリックして進みます。
![](015.png)

13. 本記事では認証モードは [Windows 認証モード] を選択し、SQL Server 管理者には [現在のユーザーの追加] ボタンをクリックして、インストール作業を実行しているユーザーを設定します。
その他の設定はそのまま   [次へ] ボタンをクリックして進みます。 
![](016.png)

14. [インストール] ボタンをクリックして、インストールを実行します。
![](017.png)

15. 完了画面に表示された [状態] がすべて [成功] になっていることを確認して [閉じる] ボタンをクリックします。
以上で SQL Server 2019 インスタンスの準備は完了です。 
 ![](018.png)

## 3. SQL Server 2019 の累積的な更新プログラム ( CU18 ) を適用する。 
1. 更新プログラムのインストーラーを選択して、右クリック メニューの [管理者として実行] をクリックします。
![](019.png)

2. ライセンス条項を確認して、 [次へ] ボタンをクリックして進みます。
![](020.png)

3. MSSQLSERVER にチェックが付いていることを確認して、 [次へ] ボタンをクリックして進みます。
![](021.png)

4. ファイルの確認が完了したら、 [次へ] ボタンをクリックして進みます。
![](022.png)

5. [更新] ボタンをクリックして、更新プログラムのインストールを実行します。
![](023.png)

6. 完了画面に表示された [状態] がすべて [成功] になっていることを確認して [閉じる] ボタンをクリックします。
以上で SCOM 用の データーベースの準備は完了です。 
![](024.png)

※ 以下の警告が表示された場合は、更新プログラムのインストール完了後、一度 Windows Server を再起動します。
![](025.png)

==================<span style="font-size: 80%;">
【補足】
検証用の SCOM 管理サーバーの最短構築に際しては必須ではありませんが、SQL Server の管理に有用な **SQL Server Management Studio (SSMS)** はインストールされることを推奨します。
SSMS は、[SQL Server Management Studio (SSMS) のダウンロード](https://learn.microsoft.com/ja-jp/sql/ssms/download-sql-server-management-studio-ssms?redirectedfrom=MSDN&view=sql-server-ver16#download-ssms) からダウンロードできます。
</span>==================


# SCOM 管理サーバーと SCOM コンソールのインストール
データーベースの準備が完了したら、いよいよ SCOM 管理サーバーと SCOM コンソールのインストールです。
詳細な手順については、以下のマニュアル サイトをご参照ください。
・[Operations Manager 管理サーバーをインストールする方法](https://learn.microsoft.com/ja-jp/system-center/scom/deploy-install-mgmt-server?view=sc-om-2022)
・[オペレーション コンソールをインストールする方法](https://learn.microsoft.com/ja-jp/system-center/scom/deploy-install-ops-console?view=sc-om-2022)

SCOM 管理サーバーと SCOM コンソールのインストール手順は以下の通りです。

## 1. SCOM 2022 のインストール モジュールを準備
1. Microsoft Evaluation Center の [System Center 2022 | Microsoft Evaluation Center](https://www.microsoft.com/ja-JP/evalcenter/download-system-center-2022) からインストール ファイル [SCOM_2022.exe] をダウンロードします。 
![](026.png)

2. ダウンロードしたファイルを SCOM をインストールする Windows Server 内にコピーします。
![](027.png)

## 2. インストール ファイルを解凍する
1. SCOM をインストールする Windows Server にドメイン管理者でサインインします。
2. SCOM 2022 のインストーラーを選択して、右クリック メニューの [管理者として実行] をクリックします。
![](028.png)

3. インストール ファイル解凍ウィザードが開きますので、[Next] ボタンをクリックして進みます。
![](029.png)

4. ライセンス条項を確認して、 [Next] ボタンをクリックして進みます。
![](030.png)

5. インストール ファイルの解凍先を確認して、 [Next] ボタンをクリックして進みます。
![](031.png)

6. [Extract] ボタンをクリックして解凍を実行します。
![](032.png)

7. [Finish] ボタンをクリックして、ウィザードを閉じます。
![](033.png)
[C:\System Center Operations Manager] フォルダ配下にインストール用ファイルが展開されていることが確認できます。 
![](034.png)

## 3. SCOM 管理サーバーと SCOM コンソールのインストール
1. SCOM をインストールする Windows Server にドメイン管理者でサインインします。
2. [C:\System Center Operations Manager] フォルダ配下の [Setup.exe] を選択して、右クリック メニューの [管理者として実行] をクリックします。
![](035.png)

3. インストール ウィザードが起動するので、[インストール] をクリックします。
![](036.png)

4. インストールする機能の選択画面で [Management サーバー] と [オペレーション コンソール] にチェックを入れて、[次へ] ボタンをクリックして進みます。
[Management サーバー] が SCOM 管理サーバー、[オペレーション コンソール] が SCOM コンソールです。
オプション機能の [Web コンソール] と [レポート サーバー] は本記事では外します。
![](037.png)

5. インストール先フォルダを指定して、[次へ] ボタンをクリックして進みます。
標準のインストール先は [C:\Program Files\Microsoft System Center\Operations Manager] です。
![](051.png)

6. 前提条件の確認画面でエラーが表示されていないことを確認して、[次へ] ボタンをクリックして進みます。
![](038.png)
※ Windows Update の実施後など、サーバーが再起動を控えている状態の場合、Windows の再起動を促す警告が表示されます。
警告を無視して [次へ] ボタンをクリックしてそのまま進めても問題なくインストールが完了する場合もありますが、Windows の再起動を促す警告が表示された場合は、念のため Windows の再起動を行ってください。

7. 管理グループ名を入力して、[次へ] ボタンをクリックして進みます。
ここでは管理グループ名を [SCOM2022] とします。
![](039.png)

8. ライセンス条項を確認して、 [次へ] ボタンをクリックして進みます。
![](040.png)

9. これから SQL Server 上に SCOM で使用するデーターベースを 2 つ構成します。
1 つ目は *オペレーション データベース (データーベース名：OperationsManager)* です。
既定のインスタンスとして SQL Server をインストールしましたので、[ サーバー名とインスタンス名] にはサーバーのコンピューター名だけを入力して、 [次へ] ボタンをクリックして進みます。
![](041.png)

10. 次に*データ ウェアハウス データベース  (データーベース名：OperationsManagerDW)* の構成を入力します。
既定のインスタンスとして SQL Server をインストールしましたので、[ サーバー名とインスタンス名] にはサーバーのコンピューター名だけを入力して、 [次へ] ボタンをクリックして進みます。
![](042.png)

11. [Operations Manager アカウントの構成] 画面でアカウント情報を入力して、 [次へ] ボタンをクリックして進みます。
本記事では、サーバーの管理者でかつ SQL Server インストール時に SQL Server の管理者に設定したドメイン管理者を指定しました。
![](043.png)
※ ドメイン管理者を指定した場合、以下の警告が表示されます。
![](044.png)
今回は検証環境構築が目的のため [OK] ボタンをクリックしてこのまま進めますが、以下のマニュアルサイトをご参考に必要に応じて SCOM 専用のアカウントをご準備ください。
　**ご参考：**[System Center Operations Manager の展開 | 必要なアカウント](https://learn.microsoft.com/ja-jp/system-center/scom/deploy-overview?view=sc-om-2022#operations-manager-administrators-role-assignment)

12. [診断と使用状況のデータ] を確認して、 [次へ] ボタンをクリックして進みます。
![](045.png)

13. 本記事では [Microsoft Update] は [オフ] を選択して [次へ] ボタンをクリックして進みます。
![](046.png)

14. [インストールの要約] を確認して、 [インストール] ボタンをインストールします。
![](047.png)

15. 完了画面でエラーが出ていないことを確認して、[閉じる] ボタンをクリックしてウィザードを閉じます。
評価版をインストールしましたので、ライセンス入手後に適用する方法が警告表示されます。
また、コンピューターを再起動する必要がある旨の警告が表示されますので、一度、Windows を再起動します。
![](048.png)

以上で、SCOM 管理サーバーと SCOM コンソールのインストールが完了です。 

# SCOM コンソールを起動して SCOM にログインする
1. SCOM をインストールした Windows Server にドメイン管理者でサインインします。
2. スタートメニューから [Microsoft System Center] -> [Operations Console] をクリックします。 
![](049.png)

3. SCOM コンソールにログインします。
Windows Server に SCOM のインストールで使用したドメイン管理者でサインインした場合は、自動的に SCOM コンソールにログインできます。 
![](050.png)

---

以上で、SCOM 管理サーバーを導入して、SCOM コンソールにログイン可能になりました。

このあと、[管理] 画面で監視要件に合わせて[管理パックのインポート](https://learn.microsoft.com/ja-jp/system-center/scom/manage-mp-import-remove-delete?view=sc-om-2022)を行います。
その後、監視機器を登録することで監視が開始されます。
これらの手順についても、今後ブログで紹介したいと考えております。

**ご参考：** [Operations Manager の監視シナリオ](https://learn.microsoft.com/ja-jp/system-center/scom/manage-monitoring-scenarios?view=sc-om-2022)
　　　　[Windows 上のエージェントの検出とインストール](https://learn.microsoft.com/ja-jp/system-center/scom/manage-mp-import-remove-delete?view=sc-om-2022)
　　　　[UNIX および Linux 上のエージェントの検出とインストール](https://learn.microsoft.com/ja-jp/system-center/scom/manage-deploy-crossplat-agent-console?view=sc-om-2022)




[SCOM 導入の流れ](#SCOM-導入の流れ) で記載しました通り、SCOM にはインフラ環境、監視要件によって導入が必要なオプション コンポーネントが存在します。
長期レポートを参照するために必要な 『レポート機能』 やパフォーマンス データやアラートの確認だけを行うSCOM の一般操作者が存在する場合に便利な 『Web コンソール』機能などは、多くのお客様環境でご利用いただいているコンポーネントです。
また、メンテナンス時の可用性や耐障害性を考慮して、複数台の SCOM 管理サーバーを導入して管理グループを構成することや、データーベースを SQL Server AlwaysOn 可用性グループで構成するなど、オール イン ワン以外の構成も存在します。
SCOM 環境の構成についても、要件定義や初期の設計フェーズにおいて検討いただければと存じます。


本番環境の構築においては多数の考慮事項がございますが、まずは今回の手順を進めていただくことで SCOM コンソールにログインすることができたと思います。
実際に SCOM コンソールにログインして、SCOM の画面構成、機能、操作性など検証いただければ幸いです。
