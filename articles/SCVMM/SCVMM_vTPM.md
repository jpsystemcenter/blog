---
title: SCVMM で TPM が有効な仮想マシンを管理する方法
date: 2024-09-22 09:00:00
tags:
  - SCVMM
  - HowTo
---

皆様こんにちは、System Center サポートチームの 石原 です。 
今回は、SCVMM で トラステッド プラットフォーム モジュール (TPM) が有効な仮想マシンを管理する方法について説明します。
<br>

<!-- more -->
## トラステッド プラットフォーム モジュール (TPM) について
SCVMM は Hyper-V 環境の統合管理システムですので、Hyper-V マネージャーやフェールオーバー クラスターマネージャーで設定可能な項目の多くは SCVMM コンソールから設定できます。ですが、一部の設定は SCVMM では対応しておらず、直接 Hyper-V  やフェールオーバークラスターにて設定する必要がある項目があります。トラステッド プラットフォーム モジュール (TPM) はほとんどのバージョンの SCVMM コンソールからは設定できない項目の一つです。
ただし SCVMM 2025 UR1 からは、SCVMM 側で直接 TPM の設定を行うことがサポートされました。

Windows 11 では TPM が有効であることがシステム要件になります。そのため、Windows 11 が稼働する Hyper-V ホストを SCVMM で管理する場合、TPM が有効な仮想マシンの管理が必要になります。
・[Windows 11 の仕様とシステム要件](https://www.microsoft.com/ja-jp/windows/windows-11-specifications)
![](001.png)

本記事では以下のケースについて手順を説明します。
１．[TPM が有効な仮想マシンの作成方法](#１．TPM-が有効な仮想マシンの作成方法)
２．[TPM が有効な仮想マシンの複製・テンプレート化方法](#２．TPM-が有効な仮想マシンの複製・テンプレート化方法)
３．[TPM が有効な仮想マシンのライブ マイグレーションができる環境の構築方法 (証明書方式)](#３．TPM-が有効な仮想マシンのライブ-マイグレーションができる環境の構築方法-証明書方式)
４．[SCVMM で直接設定を行う方法](#４．SCVMM-で直接設定を行う方法)


## １．TPM が有効な仮想マシンの作成方法
SCVMM コンソールから [新規仮想マシンの作成] にて Windows 11 を作成後、仮想マシンの電源を入れて Windows 11 OS のインストールを開始すると、以下のエラー画面が表示されて途中で失敗します。SCVMM による仮想マシンの新規作成では <b>**TPM が無効な仮想マシン**</b> が作成されますので、Windows 11 のシステム要件を満たさないためです。
■ Windows 11 インストール エラー画面
![](002.png)

そのため、仮想マシンの電源 ON の前に Hyper-V マネージャーにて TPM を有効にしておく必要があります。
■ Hyper-V マネージャーにて仮想マシンの TPM を有効化
![](003.png)

 Hyper-V マネージャーにて該当仮想マシンの TPM を有効にすることで、SCVMM コンソールで該当 VM の電源をオンにして Windows 11 OS のインストールを進めることができます。（※ もちろん、このまま Hyper-V マネージャーで VM の電源をオンにして Windows 11 OS のインストールを進めることでも問題ありません。）
■ TPM を有効化することで OS インストール が進められます。
![](004.png)


## ２．TPM が有効な仮想マシンの複製・テンプレート化方法
TPM が有効な仮想マシンは SCVMM コンソールの複製・テンプレート化のメニューが無効になります。
■ TPM 有効な仮想マシンの複製・テンプレート化 メニュー (※グレーアウトしています。)
![](005.png)

Hyper-V マネージャーで一度 TPM を無効化したのちに、複製・テンプレート化します。
■ Hyper-V マネージャーで TPM を無効化すると SCVMM 管理コンソールの複製・テンプレート化メニューが有効になります。
![](006.png)

<b>**注意**</b>
複製した仮想マシンやテンプレートから作成した VM は TPM が無効な状態です。Windows 11 コンピューターは TPM が有効であることがシステム要件のため、VM を電源オンする前に Hyper-V マネージャーで TPM 有効にしてください。

<b>**補足**</b>
TPM の有効/無効化は Hyper-V マネージャーの画面上での設定変更の他に Hyper-V コマンドでも実行することができます。
　無効化：[Disable-VMTPM (Hyper-V) | Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/hyper-v/disable-vmtpm?view=windowsserver2022-ps)
　有効化：[Enable-VMTPM (Hyper-V) | Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/hyper-v/enable-vmtpm?view=windowsserver2022-ps)

以下のスクリプトは Hyper-V ホスト上の VM の TPM 設定を一括で無効化/有効化するサンプルです。
※ Hyper-V ホスト上で実行してください。

■ Hyper-V ホスト上の仮想マシンの TPM を一斉に無効化
```CMD
$vmNames = Get-VM | Select-Object -ExpandProperty Name foreach ($vmName in $vmNames) {
      try {
             Disable-VMTPM -VMName $vmName -ErrorAction Stop
             Write-Output "Successfully disabled TPM for VM: $vmName"
      } catch {
             Write-Output "Failed to disable TPM for VM: $vmName. Error: $_"
      }
}
```
■ Hyper-V ホスト上の仮想マシンの TPM を一斉に有効化
```CMD
$vmNames = Get-VM | Select-Object -ExpandProperty Name foreach ($vmName in $vmNames) {
      try {
             Enable-VMTPM -VMName $vmName -ErrorAction Stop
             Write-Output "Successfully eabled TPM for VM: $vmName"
      } catch {
             Write-Output "Failed to enable TPM for VM: $vmName. Error: $_"
      }
}
```


## ３．TPM が有効な仮想マシンのライブ マイグレーションができる環境の構築方法 (証明書方式)
TPM が有効な仮想マシンをライブ マイグレーションするとエラー (10698) で失敗します。シールドされた VM をライブ マイグレーションする場合は、2 つ以上の保護されたホストをデプロイする必要があるためです。
■ ライブ マイグレーションがエラー (10698) で失敗
![](007.png)

対処方法は以下の２つです。
 <b>方法１．</b>ホスト ガーディアン サービス (HGS) サーバーに接続してホスト間でシールドされた VM を移行可能にする。
 <b>方法２．</b>Hyper-V ホストに保存されている証明書を各ホストにインポートしてホスト間でシールドされた VM を移行可能にする。

方法１については、以下のサイトに詳細な手順の記載がありますので、こちらを参照してください。
　[保護されたホスト ファブリックをデプロイする](https://learn.microsoft.com/ja-jp/system-center/vmm/deploy-guarded-host-fabric?view=sc-vmm-2022)

多数の Hyper-V ホストが存在する大規模環境では方法１の構成が推奨されますが、HGS サーバーが必要であることなど、準備が煩雑になります。Hyper-V ホストの台数が10台以下の環境では、証明書をお互いにインポートする方法２が簡潔になります。本記事では方法２について説明します。

=====================================================================
方法２：証明書を介した環境構築の詳細手順
=====================================================================
Hyper-V ホストに保存されている以下の証明書をライブマイグレーション先のホストにインポートすることで、ホスト間でシールドされた VM がライブ マイグレーション可能になります。

// 証明書
Shielded VM Signing Certificate (UntrustedGuardian) (ホスト名)
Shielded VM Encryption Certificate (UntrustedGuardian) (ホスト名)

TPM を有効にすると上記の 2 つの証明書が作成されます。これは、仮想マシンのキー保護機能を解除するために使用されます。ライブ マイグレーション先にこの証明書がない状態だと、上述のエラー (10698) が発生します。

以下の手順にて証明書のエクスポートとインポートを行います。

1. TPM を有効化した仮想マシンが稼働している Hyper-V ホストにログオンします。
2. 管理者権限のコマンド プロンプトを起動します。
3. 以下のコマンドを実行します。
```CMD
certutil -store "Shielded VM Local Certificates"
```
4. 以下の証明書のシリアル番号をそれぞれメモします。
Shielded VM Signing Certificate (UntrustedGuardian) (ホスト名)
Shielded VM Encryption Certificate (UntrustedGuardian) (ホスト名)
【実行例】
![](008.png)

5. 以下のコマンドを実行して、それぞれの証明書をエクスポートします。
※ <パスワード> は任意になります。
※ <シリアル番号> は 4. で確認した番号を指定してください。
```CMD
certutil -exportPFX -p <パスワード> "Shielded VM Local Certificates" <シリアル番号> <出力先フォルダ>\VMSigning.pfx
certutil -exportPFX -p <パスワード> "Shielded VM Local Certificates" <シリアル番号> <出力先フォルダ>\VMEncryption.pfx
```
【実行例】
![](009.png)
6. 作成された 2 ファイルを別の Hyper-V ホストへコピーします。
7. 別の Hyper-V ホストで管理者権限のコマンド プロンプトを起動します。
8. 以下を実行して、証明書をインポートします。
※ インポートの際にパスワードが要求されますので、エクスポートの際に指定したパスワードを入力ください。
```CMD
certutil -importPFX "Shielded VM Local Certificates" <配置先フォルダ>\VMSigning.pfx
certutil -importPFX "Shielded VM Local Certificates" <配置先フォルダ>\VMEncryption.pfx
```
【実行例】
![](010.png)
9. TPM を有効化した仮想マシンを証明書をインポートした Hyper-V ホストへ移行させて、正常に動作することを確認します。
![](011.png)

10. 上の 6. ～ 9. の作業をクラスターに参加しているノード (Hyper-V ホスト) 全てで実行します。お互いに証明書をインポートすることで、どの Hyper-V ホストに対してもライブ マイグレーションできるようになります。

---

SCVMM で TPM が有効な仮想マシンを管理する方法に関する説明は以上の通りです。
Windows 11 コンピューターが稼働する Hyper-V ホストを SCVMM で管理する際の参考にしていただけますと幸いです。

## ４．SCVMM で直接設定を行う方法
SCVMM 2025 UR1 では、従来では SCVMM 側では設定を行うことができなかった TPM の設定をサポートするようになりました。
[System Center 2025 Virtual Machine Manager の更新プログラムロールアップ 1](https://support.microsoft.com/ja-jp/topic/system-center-2025-virtual-machine-manager-%E3%81%AE%E6%9B%B4%E6%96%B0%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%A0%E3%83%AD%E3%83%BC%E3%83%AB%E3%82%A2%E3%83%83%E3%83%97-1-7c23151b-26b5-4e1f-9e2a-00a4cfb5748f)
```
新しい VM の作成時と既存の VM の構成の変更中に vTPM を有効にするサポート
```
SCVMM 2022 ではまだ TPM 設定サポートは追加されておりません (2026 年 6 月 8 日時点) ので、現状では SCVMM 2025 ユーザー様向けの機能としてのご紹介になります。
当然ですが、TPM の設定は第 2 世代の仮想マシンに限定されます。Hyper-V であっても、第 1 世代の仮想マシンに対して TPM の設定を行うことはできません。

=====================================================================
方法１：SCVMM コンソールを使用した設定手順
=====================================================================
仮想マシンの新規作成時や既存仮想マシンの設定変更時、[ハードウェア構成] -> [セキュリティ] の配下に "トラステッド プラットフォーム モジュールを有効にする" の設定が追加されました。
こちらのチェックボックスを選択することで、TPM の有効化が可能です。
![](012.png)
もちろん、こちらの設定は新規および既存のハードウェア プロファイルからもアクセス可能です。
![](013.png)

ご参考情報とはなりますが、第 1 世代では以下のように、TPM の設定自体が表示されない動作となります。
Hyper-V マネージャーを使用した場合も同様となりますので、このような動作となった場合は仮想マシンの世代が第 1 世代である、と判断いただけます。
![](014.png)

=====================================================================
方法２：コマンドを使用した設定手順
=====================================================================
2026 年 6 月 8 日時点では公開情報がまだアップデートされていませんが、仮想マシンやハードウェア プロファイルの設定に関わるコマンドにおいて、下記のパラメーターを指定いただけるようになりました。
```
-TPMEnabled

型: Boolean
$True 指定の場合: TPM を有効化
$False 指定の場合: TPM を無効化
```

下記にコマンドを実行する際の例を紹介します。
例 1: 既存の仮想マシンに対して TPM を有効化するコマンド
```
Get-SCVirtualMachine -VMMServer <SCVMM サーバーの FQDN> -Name "<仮想マシン名>" | set-scvirtualmachine -TPMEnabled $True
```

例 2: 既存のハードウェア プロファイルに対して TPM を無効化するコマンド
```
$HardwareProfile = Get-SCHardwareProfile -VMMServer <SCVMM サーバーの FQDN> | where {$_.Name -eq "<ハードウェア プロファイル名>"}
Set-SCHardwareProfile -HardwareProfile $HardwareProfile -TPMEnabled $false
```

SCVMM コンソールで操作を行う際、[スクリプトの表示] をクリックすることで、内部上どんなスクリプトが実行されているかチェックできます。
上記は非常に簡単なコマンドになりますので、ご要望に応じてお客様の方でもスクリプトをチェックいただき、ご要望に合わせてスクリプトをカスタマイズすることをお試しください。