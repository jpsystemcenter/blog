---
title: SCOM 2016 から SCOM 2022 へ移行 or アップグレードについて
date: 2022-08-18 13:00:00
tags:
  - SCOM
  - アップグレード
  - 新環境への移行
---

<!-- more -->
皆様こんにちは、System Center サポートチームの 佐藤 です。
本日は System Center Operations Manager（以後 SCOM）  の新環境への移行やアップグレードについての考え方をご紹介いたします。

まず前提として SCOM の仕様内容について以下記述いたします。
1. SCOM 管理サーバーのバージョン違いの共存について
　基本的には 1 バージョン前の最新の UR とのみ共存が可能となります。
　参考：[共存の要件](https://docs.microsoft.com/ja-jp/system-center/scom/system-requirements?view=sc-om-2022#supported-coexistence)

2. SCOM 管理サーバーのシステム要件
　SCOM 管理サーバーにはバージョンごとにシステム要件がございます。
　　[SCOM 2022 のシステム要件](https://docs.microsoft.com/ja-jp/system-center/scom/system-requirements?view=sc-om-2022#supported-coexistence)
　　[SCOM 2019 のシステム要件](https://docs.microsoft.com/ja-jp/system-center/scom/system-requirements?view=sc-om-2019#supported-coexistence)
　　[SCOM 2016 のシステム要件](https://docs.microsoft.com/ja-jp/system-center/scom/system-requirements?view=sc-om-2016#supported-coexistence)
　　[SCOM 2012 R2のシステム要件](https://docs.microsoft.com/ja-jp/previous-versions/system-center/system-center-2012-r2/dn281935(v=sc.12))

上記ページを参照いただくと確認できるように 2 バージョン 離れていると要件を満たす OS や SQL Server で共有するバージョンがございません。
例）
SCOM 2022 に対応する OS バージョン
　Windows Server 2019 Standard/Datacenter
　Windows Server 2019 Server Core
　Windows Server 2022 Standard/Datacenter
　Windows Server 2022 Server Core
SCOM 2016 に対応する OS バージョン
　Windows Server 2012 R2  Standard/Datacenter
　Windows Server 2016 Standard/Datacenter
　Windows 2016 Server Core

一方、多くのお客様が HW の EOL 対応や減価償却期間を踏まえたリプレース、もしくは Microsoft サポートフェーズを踏まえた移行やアップグレードを検討する場合、新環境は 2 バージョン以上先になると考えております。
その場合 2 段階のアップグレード （例、SCOM 2016 -> SCOM 2019 -> SCOM 2022）することはシステム要件を踏まえると現実的（※1）ではございません。

そのため選択肢としては新規に SCOM 管理サーバーを構築したのちに、監視対象サーバーから旧エージェントをアンインストールいただき、新バージョンをインストールして入れ替えることなります。
アンインストール/インストール方法については下記公開情報をご参照ください。

アンインストール方法の関連情報（SCOM 2016 に関する公開情報）
　[Windows ベースのコンピューターからエージェントをアンインストールする](https://docs.microsoft.com/ja-jp/system-center/scom/manage-uninstall-windows-agent?view=sc-om-2016)
　[Upgrading and Uninstalling Agents on UNIX and Linux Computers (UNIX および Linux コンピューター上のエージェントのアップグレードおよびアンインストール)](https://docs.microsoft.com/ja-jp/system-center/scom/manage-upgrade-uninstall-crossplat-agent?view=sc-om-2016#uninstalling-agents)
　[Manually Uninstalling Agents from UNIX and Linux Computers](https://docs.microsoft.com/ja-jp/system-center/scom/manage-uninstall-crossplat-agent?view=sc-om-2016)
インストール方法の関連情報（SCOM 2022 に関する公開情報）
　[検出ウィザードを使ってエージェントを Windows にインストールする](https://docs.microsoft.com/ja-jp/system-center/scom/manage-deploy-windows-agent-console?view=sc-om-2022)
　[MOMAgent.msi を使用して Windows エージェントを手動でインストールする](https://docs.microsoft.com/ja-jp/system-center/scom/manage-deploy-windows-agent-manually?view=sc-om-2022)
　[検出ウィザードを使ってエージェントを UNIX および Linux にインストールする](https://docs.microsoft.com/ja-jp/system-center/scom/manage-deploy-crossplat-agent-console?view=sc-om-2022)
　[コマンド ラインから UNIX および Linux コンピューターにエージェントをインストールする](https://docs.microsoft.com/ja-jp/system-center/scom/manage-install-crossplat-agent-cmdline?view=sc-om-2022)

また新環境を構築することで以下のメリットがございます。
　・旧環境と併設できるため、エージェント側の状況（監視を停止できる時間）を踏まえて移行できる
　・アップグレードの場合サポートしていない管理パックが引き継がれるリスクが混在するが、その問題を回避できる
　・新バージョンでサポートしていない監視対象については、引き続き旧環境で監視できる
　　　例） SCOM 2022 で監視対象となる Windows のバージョン（※2）は Windows Server 2012 以降/ Windows 10 以降となります。

新環境への移行やアップグレードについてお困りごとがございましたら Microsoft までお問合せくださいませ。

※ 1
OS や SQL Server のバージョンアップや 要件を満たす HW へ移行をすることで実現できる可能性はございます。
※ 2
[監視対象の要件](https://docs.microsoft.com/ja-jp/system-center/scom/system-requirements?view=sc-om-2022#microsoft-monitoring-agent-operating-system)

※本情報の内容（添付文書、リンク先などを含む）は、作成日時点でのものであり、予告なく変更される場合があります。 