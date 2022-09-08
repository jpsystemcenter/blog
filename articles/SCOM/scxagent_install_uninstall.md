---
title: Linux エージェントのインストール/アンインストール方法について
date: 2022-09-08 11:00:00
tags:
  - SCOM
  - HowTo
  - SCXAgent
---

<!-- more -->
皆様こんにちは、System Center サポートチームの 佐藤 です。
本日は SCOM サーバーで監視できる UNIX/Linux 監視方法や監視を構成しているエージェント側のコンポーネント、ならびに具体的なインストール/アンインストール方法についてご紹介いたします。

## 監視対象とエージェントのコンポーネントについて

### 監視対象

まず各 SCOM バージョンで監視対象にできる UNIX/Linux の OS については下記当社公開ドキュメントに記載しておりますのでご参照ください。

|バージョン|確認先 URL|
|------------|------------|
| 2016 | [サポートされている UNIX/Linux の OS ](https://docs.microsoft.com/ja-jp/system-center/scom/plan-supported-crossplat-os?view=sc-om-2016) |
| 2019 | [サポートされている UNIX/Linux の OS ](https://docs.microsoft.com/ja-jp/system-center/scom/plan-supported-crossplat-os?view=sc-om-2019) |
| 2022 | [サポートされている UNIX/Linux の OS ](https://docs.microsoft.com/ja-jp/system-center/scom/plan-supported-crossplat-os?view=sc-om-2022) |

### エージェントのコンポーネント

監視対象に UNIX/Linux に SCOM エージェント（以後、SCXAgent）をインストールしますと下記ディレクトリ、ならびにその配下にファイルが生成されます。

|ディレクトリパス|内容|
|------------|------------|
| /opt/omi/ | Open Management Infrastructure (OMI) は次のディレクトリにインストールされます |
| /opt/microsoft/scx/| UNIX/Linux エージェントは次のディレクトリにインストールされます |
| /var/opt/microsoft/scx/log/ | UNIX/Linux エージェントは次のディレクトリにログ ファイルを保持します |
| /var/opt/omi/log/ | OMI は次のディレクトリにログ ファイルを保持します |
| /etc/opt/microsoft/scx/ | 証明書などのエージェントの構成ファイルは次のディレクトリに格納されます |
| /etc/opt/omi/ | OMI 構成ファイルは次のディレクトリに格納されます |


## インストール方法とアンインストール方法について

### インストール方法

ここからは具体的なインストール方法は記載いたします。

まず監視をするため UNIX / Linux 管理パックを SCOM 管理サーバーにインポートする必要がございます。
以下ダウンロード先から対象管理パックをダウンロードいただき、SCOM 管理サーバーに [ インポート ]（https://docs.microsoft.com/ja-jp/system-center/scom/manage-mp-import-remove-delete?view=sc-om-2019）します。

|バージョン|確認先 URL|
|------------|------------|
| 2016 | [ ダウンロード先 ](https://www.microsoft.com/en-us/download/details.aspx?id=29696) |
| 2019 / 2022 | [ ダウンロード先 ](https://www.microsoft.com/en-us/download/details.aspx?id=58208) |


SCXAgent のインストールは以下２つの方法がございます。
#### 1. SCOM Operations Console 画面での操作
手順は[ 当ページ ]（https://docs.microsoft.com/ja-jp/system-center/scom/manage-deploy-crossplat-agent-console?view=sc-om-2019#to-discover-and-install-an-agent-on-a-unix-or-linux-computer）に記載がございます。
下記を確認できる事から可能な限りこちらの操作方法でのインストールを推奨しております。
　・SCOM 管理サーバーに必要な管理パックが入っているか
　・エージェントコンピューターと通信できる環境となっているか
　・インストール時に指定する Linux のユーザーが使用できるか

 
#### 2. Linux のローカルでの操作
手順は [ 当ページ ]（https://docs.microsoft.com/en-us/system-center/scom/manage-install-crossplat-agent-cmdline?view=sc-om-2019）に記載がございます。
CentOS を例に記載しますと、上記ページの下記箇所のコマンドを実行いただくことでインストールできます。
- 事前作業としてエージェントのインストールシェル （ scx-<version>.universalr.<version>.<arch>.sh ）を事前に対象コンピューターに任意フォルダに格納します。
   エージェント パッケージは、管理サーバーの次のフォルダーにあります。
　%ProgramFiles%\Microsoft System Center\Operations Manager\Server\AgentManagement\UnixAgents\DownloadedKits

- 以下コマンドでパッケージをインストールします
　sh ./scx-<version>.universalr.<version>.<arch>.sh --install --enable-opsmgr
- 以下コマンドでパッケージのインストール結果を確認します。コマンド結果としてアウトプットが出力されればインストールされております。
　rpm -q scx
- 以下コマンドで SCX Server の実行状況を確認します。
　scxadmin -status
　
(結果例) omiagent については実行した後ある程度時間が経過したのち 1 instance 以上が実行状態となります。
omiserver: is running
omiagent: is stopped
または
omiserver: is running
omiagent: XX instance running

### アンインストール方法

#### 1. SCOM Operations Console 画面での操作
手順は [ 当ページ ]（https://docs.microsoft.com/ja-jp/system-center/scom/manage-upgrade-uninstall-crossplat-agent?view=sc-om-2019#uninstalling-agents）に記載がございます。


#### 2. Linux のローカルでの操作
手順は [ 当ページ ] （https://docs.microsoft.com/en-us/system-center/scom/manage-uninstall-crossplat-agent?view=sc-om-2016）に記載がございます。
CentOS を例に記載しますと、上記ページの下図箇所のコマンドを実行いただくことでアンインストールできます。
- rootユーザでログオンし、以下コマンドでパッケージをアンインストールします
　rpm -e scx
- パッケージがアンインストールされたことを確認するには、次のように入力します。何も出力されなければアンインストールできております。
　rpm -q scx
