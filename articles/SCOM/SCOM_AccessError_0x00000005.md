---
title: AdtAdmin / setquery でアクセス拒否エラー(0x00000005)が発生する場合の対処法
date: 2022-01-06 12:00:00
tags:
  - SCOM
  - Troubleshoot
  - Audit Collection Service (ACS)
disableDisclaimer: false
---

<!-- more -->
皆様こんにちは、System Center サポートチームの 佐藤 です。

本日は System Center Operations Manager（以後 SCOM） の機能である 監査コレクション サービス (ACS) （[監査コレクション サービス (ACS) コレクターとデータベースをインストールする方法 | Microsoft Docs](https://docs.microsoft.com/ja-jp/system-center/scom/deploy-install-acs?view=sc-om-2019)）において WQL クエリ設定を実行する際に必要な設定をご紹介いたします。

ACS は WQL（[WMI Provider for Server Events と WQL の使用 - SQL Server | Microsoft Docs](https://docs.microsoft.com/ja-jp/sql/relational-databases/wmi-provider-server-events/using-wql-with-the-wmi-provider-for-server-events?view=sql-server-ver15)）を使用して取得対象のイベントの範囲を絞ることが可能です。
これは ACS データを保存するデータベースの肥大化の防止、また不要なデータを取得しないことでSCOM 管理サーバー/監視対象サーバーの処理負荷の軽減に寄与いたします。



しかし、規定の設定では下記のように WQL を実行しようとする際にアクセス拒否エラーが出力される事が確認されております。
>■コマンド例
> ` ` `C:\Windows\System32\Security\AdtServer>AdtAdmin /setquery /query:"SELECT * FROM AdtsEvent WHERE EventId=XXX" ` ` `

その際はレジストリエディタにて以下キーに対してユーザー "NETWORK SERVICE" へ フルコントールなどの "値の設定"の権限付与することで改善されます。
こちらの設定については SCOM 用に定義されているパラーメータ設定であり、設定変更により他アプリケーションへの影響はございませんのでご安心ください。
>■キー
> ` ` `HKLM\SYSTEM\CurrentControlSet\Services\AdtServer\Parameters` ` `


参考図：

![参考図](https://user-images.githubusercontent.com/71251920/148327138-6c2ba2db-ced8-4408-9c43-144a170ee18d.png)

> 対象バージョン：
>　System Center Operations Manager 2012 R2
>　System Center Operations Manager 2016
>　System Center Operations Manager 2019
