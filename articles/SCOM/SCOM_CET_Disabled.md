---
title: SCOM エージェント サーバーの CET (Control-flow Enforcement Technology) 無効化について
date: 2024-03-26 17:00:00
tags:
  - SCOM
  - Trouble
---

皆様こんにちは、System Center サポートチームの 石原 です。
今回は、ハードウェア ベースのセキュリティ機能の CET (Control-flow Enforcement Technology) が有効な場合、SCOM エージェントのプロセスが短時間でクラッシュする既知の問題の対処方法について説明します。
<br>

<!-- more -->
## CET (Control-flow Enforcement Technology) が有効な場合の SCOM エージェントの既知の問題について
CET (Control-flow Enforcement Technology) をサポートするプロセッサを搭載したハードウェアにインストールした Windows は、<b>ハードウェア強制されたスタック保護機能 (Hardware-enforced Stack Protection) </b>を有効にすることができます。
※ 注意：ハードウェア強制されたスタック保護機能は、新しい OS ビルドで有効な機能になります。
　 Windows 10 では、20H1 (19041) and 20H2 (19042) 以降の OS ビルドでサポートされるようになっています。
　 ご参考：[2020 年 11 月 30 日 — KB4586853 (OS ビルド 19041.662 および 19042.662)](https://support.microsoft.com/ja-jp/help/4586853/windows-10-update-kb4586853)
　 <img src="001.png" width="80%">


ハードウェア強制されたスタック保護機能 (以降、本機能) は新しいセキュリティ機能であり、本記事の公開日時点の SCOM エージェントは、<b>いずれのバージョンも本機能を使用しません。</b>
そのため、SCOM エージェント用には本機能は無効で良いですが、SCOM 2022 UR2 までは、明示的に本機能を無効にする設定が SCOM エージェントに組み込まれていません。本機能をサポートする環境に SCOM 2022 UR2 より前の SCOM エージェントをインストールした場合、SCOM エージェントは本機能が有効なプロセスとして起動します。

■ 参考：SCOM エージェントのプロセス (※ SCOM エージェントのプロセスは HealthService.exe ) が
　　　　 ハードウェア強制されたスタック保護機能を有効な状態で起動した際の画面ショット ![](002.png)

<b>SCOM エージェントが、ハードウェア強制されたスタック保護機能を有効な状態で起動すると、短時間でプロセスがクラッシュして、該当サーバーが SCOM で管理できない状態になる既知の事例があります。</b>

---
## 対処方法 ① 更新プログラムのロールアップを適用
SCOM 2022 をご利用の場合、<b>更新プログラムロールアップ 2</b>にて修正されていますので、更新プログラムロールアップ 2を適用してください。更新プログラムロールアップ 2 を適用することで、ハードウェア強制されたスタック保護機能が無効化された SCOM エージェントにアップデートすることができます。

・[SCOM 2022 更新プログラムのロールアップ 2 のダウンロードと適用手順](https://support.microsoft.com/kb/5031649)　<img src="003.png" width="80%">

※ 更新プログラムのロールアップの適用手順につきましては、以下のサポートブログもご参照ください。
　[SCOM の 更新プログラムのロールアップの適用手順](../SCOM_Update_Rollup/)


## 対処方法 ② PowerShell コマンドによる対処
すぐには更新プログラムのロールアップを適用できない場合や SCOM 2019 をご利用の場合、PowerShell コマンドにて SCOM エージェント プロセスのハードウェア強制されたスタック保護機能を無効化することで回避できます。
実行手順は以下の通りです。
1. SCOM エージェントをインストールしたサーバーにログインします。
2. PowerShell を管理者で起動します。
3. 以下のコマンドで SCOM エージェント プロセスのハードウェア強制されたスタック保護機能を無効化します。
  `Set-ProcessMitigation -Name MonitoringHost.exe -Disable UserShadowStack`
4. 以下のコマンドで SCOM エージェント プロセス (HealthService.exe) を再起動します。
  `Restart-Service -Name HealthService`

---
## ご参考 (英語サイト)
ハードウェア強制されたスタック保護機能について
・[Understanding Hardware-enforced Stack Protection](https://techcommunity.microsoft.com/t5/windows-os-platform-blog/understanding-hardware-enforced-stack-protection/ba-p/1247815)
・[Developer Guidance for Hardware-enforced Stack Protection](https://techcommunity.microsoft.com/t5/windows-os-platform-blog/developer-guidance-for-hardware-enforced-stack-protection/ba-p/2163340)

