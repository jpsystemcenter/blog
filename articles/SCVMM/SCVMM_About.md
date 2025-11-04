---
title: SCVMM の機能紹介
date: 2025-11-04 09:00:00
tags:
  - SCVMM
  - HowTo
---

皆様こんにちは、System Center サポートチームの 保科 です。 
今回は、はじめて SCVMM をご利用いただく方に向けて、SCVMM とは何ができるのか？という点を説明します。
<br>

<!-- more -->
## 概要
昨今の VMWare ライセンス費用の実質的な高騰に伴い、多くのお客様が Hyper-V への移行をご検討いただいているものと思います。
Hyper-V における VMWare vCenter の役割を果たすのが SCVMM となります。
また、Microsoft 公式として、VMWare 環境から Hyper-V 環境へ仮想マシンを変換することをサポートするアプリケーションは、現在 SCVMM のみとなります。
この移行を行うために、大変ありがたいことに多くのお客様に SCVMM をご利用いただいております。
しかし、SCVMM を導入したとして、そもそも SCVMM では何ができるのか？どんな機能があるのか？vCenter の代替になりえるのか？といったご質問を多くいただいております。
本記事では、現在移行をご検討いただいている方に向けて、SCVMM の機能概要を説明するものとなります。
概要の紹介にとどまりますが、今後移行を予定されている方も、移行が完了された後にどんな機能があるか知りたい方も、ぜひこちらの内容をご参考としてください。

## SCVMM で実現可能な機能
SCVMM をご利用いただくことで、以下の利用が可能です。
まず SCVMM の概要を紹介している弊社公開情報をご案内いたします。
[Virtual Machine Manager とは | Microsoft Learn](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Flearn.microsoft.com%2Fja-jp%2Fsystem-center%2Fvmm%2Foverview%3Fview%3Dsc-vmm-2022&data=05%7C02%7CAtsushi.Hoshina%40microsoft.com%7Cdd62472f8425493f517a08dd62bc113a%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C638775282115746627%7CUnknown%7CTWFpbGZsb3d8eyJFbXB0eU1hcGkiOnRydWUsIlYiOiIwLjAuMDAwMCIsIlAiOiJXaW4zMiIsIkFOIjoiTWFpbCIsIldUIjoyfQ%3D%3D%7C0%7C%7C%7C&sdata=TE%2BmmNN7Gz%2B4Qu92qMHfjYrKrkds44UdwsBVUSP17X0%3D&reserved=0)
こちらの内容を、以下の通りに要約いたします。
- 各種 Hyper-V ホストやクラスター、VMWare ESXi ホストやクラスターの管理、およびそれらの上で動作する仮想マシンの管理:
SCVMM では、ご利用の環境内に存在するホストやクラスター、およびそれらの上で動作する仮想マシンを、単一の SCVMM ファブリック内で一元管理可能です。
Hyper-V に関しては、基本的に Hyper-V マネージャーで設定可能な項目は SCVMM 内でも設定可能です。
もちろん、仮想マシンの展開も SCVMM より実施いただけます。
- ネットワーク設定の管理:
ホストや仮想マシンに必要なネットワーク設定を管理することが可能です。
[VMM ネットワーク ファブリックのセットアップ | Microsoft Learn](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Flearn.microsoft.com%2Fja-jp%2Fsystem-center%2Fvmm%2Fmanage-networks%3Fview%3Dsc-vmm-2022&data=05%7C02%7CAtsushi.Hoshina%40microsoft.com%7Cdd62472f8425493f517a08dd62bc113a%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C638775282115755432%7CUnknown%7CTWFpbGZsb3d8eyJFbXB0eU1hcGkiOnRydWUsIlYiOiIwLjAuMDAwMCIsIlAiOiJXaW4zMiIsIkFOIjoiTWFpbCIsIldUIjoyfQ%3D%3D%7C0%7C%7C%7C&sdata=batbjHxt57YTLZkbySzDH2HU8QDBfq08ct7efOirfUA%3D&reserved=0)
代表的な機能をご紹介します。
    - 論理ネットワークとその上に IP アドレス プールを設定することで、その IP アドレス プール範囲内から、展開される仮想マシンに IP アドレスを自動付与
    - 論理スイッチを作成することで、指定した設定に沿ってホストの NIC チーミングを SCVMM 経由で設定
    - ストレージ機器の管理:
SCVMM 上でストレージ機器を管理することが可能です。
管理対象のストレージ機器に共有フォルダを作成したり、ストレージ機器上の LUN をホストに割り当てすることが可能です。
これらのストレージ機器で作成された各種設定事項は、ホスト上で仮想マシンの配置に使用することが可能です。
[VMM 記憶域ファブリックを設定する | Microsoft Learn](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Flearn.microsoft.com%2Fja-jp%2Fsystem-center%2Fvmm%2Fmanage-storage%3Fview%3Dsc-vmm-2022&data=05%7C02%7CAtsushi.Hoshina%40microsoft.com%7Cdd62472f8425493f517a08dd62bc113a%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C638775282115767042%7CUnknown%7CTWFpbGZsb3d8eyJFbXB0eU1hcGkiOnRydWUsIlYiOiIwLjAuMDAwMCIsIlAiOiJXaW4zMiIsIkFOIjoiTWFpbCIsIldUIjoyfQ%3D%3D%7C0%7C%7C%7C&sdata=Mv9xRkDWnxndxKqf8DtJ2DRnflCxJ6Ius%2F6bZHl5X60%3D&reserved=0)
- プライベート クラウドの設定:
SCVMM サーバー上にプライベート クラウドの設定を行うことで、各種物理リソースをクラウドに割り当てることが可能です。
割り当てられた物理リソースが利用可能な限り、ユーザーや仮想マシンが配置される物理リソースの設定を行うことなく、仮想マシンの管理運用が可能となります。
[プライベート VMM クラウドを展開する | Microsoft Learn](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Flearn.microsoft.com%2Fja-jp%2Fsystem-center%2Fvmm%2Fdeploy-cloud%3Fview%3Dsc-vmm-2022&data=05%7C02%7CAtsushi.Hoshina%40microsoft.com%7Cdd62472f8425493f517a08dd62bc113a%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C638775282115775739%7CUnknown%7CTWFpbGZsb3d8eyJFbXB0eU1hcGkiOnRydWUsIlYiOiIwLjAuMDAwMCIsIlAiOiJXaW4zMiIsIkFOIjoiTWFpbCIsIldUIjoyfQ%3D%3D%7C0%7C%7C%7C&sdata=kS4%2FlC0G7cygkejvyhfB6v6MG8rMgSdANGSu1tmlkwE%3D&reserved=0)
- インフラストラクチャ サーバーの管理:
SCVMM では、仮想マシンで使用する ISO ファイルや vhdx ファイル等を配置するために使用可能なライブラリ サーバーという機能がございます。
これらに各種仮想マシンで使用するファイルを配置いただくことで、例えばライブラリ サーバーに配置された ISO ファイルを、管理対象の仮想マシンすべてに割り当てることが可能となります。
その他にも PXE サーバー、IPAM サーバー、WSUS 等を管理対象とすることができ、それらのサーバー毎に提供される機能をホストや仮想マシンの管理に役立てることが可能です。
[VMM コンピューティング ファブリックでインフラストラクチャ サーバーを設定する | Microsoft Learn](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Flearn.microsoft.com%2Fja-jp%2Fsystem-center%2Fvmm%2Finfrastructure-server%3Fview%3Dsc-vmm-2022&data=05%7C02%7CAtsushi.Hoshina%40microsoft.com%7Cdd62472f8425493f517a08dd62bc113a%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C638775282115782734%7CUnknown%7CTWFpbGZsb3d8eyJFbXB0eU1hcGkiOnRydWUsIlYiOiIwLjAuMDAwMCIsIlAiOiJXaW4zMiIsIkFOIjoiTWFpbCIsIldUIjoyfQ%3D%3D%7C0%7C%7C%7C&sdata=DifC7BvzG2drcXDEKVKUE%2FEH6Qc9AKP993VshFkX4v4%3D&reserved=0)
[VMM コンピューティング ファブリックで更新サーバーを設定する | Microsoft Learn](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Flearn.microsoft.com%2Fja-jp%2Fsystem-center%2Fvmm%2Fupdate-server%3Fview%3Dsc-vmm-2022&data=05%7C02%7CAtsushi.Hoshina%40microsoft.com%7Cdd62472f8425493f517a08dd62bc113a%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C638775282115789456%7CUnknown%7CTWFpbGZsb3d8eyJFbXB0eU1hcGkiOnRydWUsIlYiOiIwLjAuMDAwMCIsIlAiOiJXaW4zMiIsIkFOIjoiTWFpbCIsIldUIjoyfQ%3D%3D%7C0%7C%7C%7C&sdata=3hX6uvRbS6v6LUnhi5sQH73bXAlrT9fAtt%2BStcDozkc%3D&reserved=0)
- 仮想化ホストの追加、プロビジョニング、管理:
SCVMM を使用することで、ベアメタル コンピューターより Hyper-V ホストやクラスターを展開することが可能です。
また、管理対象のスタンドアロン Hyper-V ホストを使用し、SCVMM よりそれらのホストでクラスターを新規に構成することも可能です。
クラスターの OS をアップグレードいただく場合、SCVMM を使用することで、ベアメタル展開の仕様に準拠してローリング アップグレードが可能となります。
[ベア メタル コンピューターから Hyper-V ホストまたはクラスターをプロビジョニングする | Microsoft Learn](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Flearn.microsoft.com%2Fja-jp%2Fsystem-center%2Fvmm%2Fhyper-v-bare-metal%3Fview%3Dsc-vmm-2022&data=05%7C02%7CAtsushi.Hoshina%40microsoft.com%7Cdd62472f8425493f517a08dd62bc113a%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C638775282115797346%7CUnknown%7CTWFpbGZsb3d8eyJFbXB0eU1hcGkiOnRydWUsIlYiOiIwLjAuMDAwMCIsIlAiOiJXaW4zMiIsIkFOIjoiTWFpbCIsIldUIjoyfQ%3D%3D%7C0%7C%7C%7C&sdata=DGzxiC2IYhzkwYy7nMQvaqle%2B%2FbZYVSdhinY%2FBprHYk%3D&reserved=0)
[VMM ファブリックの Hyper-V スタンドアロン ホストからクラスターをプロビジョニングする | Microsoft Learn](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Flearn.microsoft.com%2Fja-jp%2Fsystem-center%2Fvmm%2Fhyper-v-standalone%3Fview%3Dsc-vmm-2022&data=05%7C02%7CAtsushi.Hoshina%40microsoft.com%7Cdd62472f8425493f517a08dd62bc113a%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C638775282115804669%7CUnknown%7CTWFpbGZsb3d8eyJFbXB0eU1hcGkiOnRydWUsIlYiOiIwLjAuMDAwMCIsIlAiOiJXaW4zMiIsIkFOIjoiTWFpbCIsIldUIjoyfQ%3D%3D%7C0%7C%7C%7C&sdata=jFpP4KKxVSAEgJX0EZLMr%2Fcd3hVjuZbsDKZJ%2Fy5j%2BUw%3D&reserved=0)
[VMM で Hyper-V ホスト クラスターの Windows Server へのローリング アップグレードを実行する | Microsoft Learn](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Flearn.microsoft.com%2Fja-jp%2Fsystem-center%2Fvmm%2Fhyper-v-rolling-upgrade%3Fview%3Dsc-vmm-2022&data=05%7C02%7CAtsushi.Hoshina%40microsoft.com%7Cdd62472f8425493f517a08dd62bc113a%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C638775282115811448%7CUnknown%7CTWFpbGZsb3d8eyJFbXB0eU1hcGkiOnRydWUsIlYiOiIwLjAuMDAwMCIsIlAiOiJXaW4zMiIsIkFOIjoiTWFpbCIsIldUIjoyfQ%3D%3D%7C0%7C%7C%7C&sdata=XLhAatob%2FiWgIuGnnOH0%2BHGoRcQiy58ZAjBps7zDBfI%3D&reserved=0)
- VM テンプレートの利用:
既存の仮想マシンを VM テンプレート化することで、テンプレート化された時点の OS 設定で仮想マシンを複数デプロイすることが可能です。
テンプレートはライブラリに保存いただく必要がございます。
[VM テンプレートを VMM ライブラリに追加する | Microsoft Learn](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Flearn.microsoft.com%2Fja-jp%2Fsystem-center%2Fvmm%2Flibrary-vm-templates%3Fview%3Dsc-vmm-2022&data=05%7C02%7CAtsushi.Hoshina%40microsoft.com%7Cdd62472f8425493f517a08dd62bc113a%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C638775282115818033%7CUnknown%7CTWFpbGZsb3d8eyJFbXB0eU1hcGkiOnRydWUsIlYiOiIwLjAuMDAwMCIsIlAiOiJXaW4zMiIsIkFOIjoiTWFpbCIsIldUIjoyfQ%3D%3D%7C0%7C%7C%7C&sdata=i8KJgOIGtAKgR5q%2BPhDffzeEZzfGlUjXR%2BmISq08nEA%3D&reserved=0)
[VMM ライブラリにサービス テンプレートを追加する | Microsoft Learn](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Flearn.microsoft.com%2Fja-jp%2Fsystem-center%2Fvmm%2Flibrary-service-templates%3Fview%3Dsc-vmm-2022&data=05%7C02%7CAtsushi.Hoshina%40microsoft.com%7Cdd62472f8425493f517a08dd62bc113a%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C638775282115825118%7CUnknown%7CTWFpbGZsb3d8eyJFbXB0eU1hcGkiOnRydWUsIlYiOiIwLjAuMDAwMCIsIlAiOiJXaW4zMiIsIkFOIjoiTWFpbCIsIldUIjoyfQ%3D%3D%7C0%7C%7C%7C&sdata=IzogSQgeIvXzzJyNLpCkWw1o5D315e5f47tPqYWnZZk%3D&reserved=0)
- Azure 管理:
SCVMM は、Azure Arc 対応 SCVMM として Azure に登録いただけます。
登録を行うことで、SCVMM 上に存在する各種リソースを Azure で管理いただけるようになります。
また、テンプレートを使用して SCVMM プライベート クラウド上に仮想マシンを展開いただけるようにもなります。
[Azure Arc 対応 System Center Virtual Machine Manager の概要 - Azure Arc | Microsoft Learn](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Flearn.microsoft.com%2Fja-jp%2Fazure%2Fazure-arc%2Fsystem-center-virtual-machine-manager%2Foverview%23how-does-it-work&data=05%7C02%7CAtsushi.Hoshina%40microsoft.com%7Cdd62472f8425493f517a08dd62bc113a%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C638775282115834558%7CUnknown%7CTWFpbGZsb3d8eyJFbXB0eU1hcGkiOnRydWUsIlYiOiIwLjAuMDAwMCIsIlAiOiJXaW4zMiIsIkFOIjoiTWFpbCIsIldUIjoyfQ%3D%3D%7C0%7C%7C%7C&sdata=0mUGNAK4MlLir4dZTCM7J0fRMfm0SuADj9G0R0vF%2BjQ%3D&reserved=0)
- VMWare 仮想マシンを Hyper-V 仮想マシンに変換:
SCVMM では、VMWare ESXi ホストの管理だけでなく、その上で動作する仮想マシンを Hyper-V 仮想マシンに変換いただけます。
現状弊社公式より提供されているアプリケーションにおいて、この変換が実施可能なアプリケーションは SCVMM のみとなります。
[VMM ファブリックで VMware VM を Hyper-V に変換する | Microsoft Learn](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Flearn.microsoft.com%2Fja-jp%2Fsystem-center%2Fvmm%2Fvm-convert-vmware%3Fview%3Dsc-vmm-2022&data=05%7C02%7CAtsushi.Hoshina%40microsoft.com%7Cdd62472f8425493f517a08dd62bc113a%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C638775282115841828%7CUnknown%7CTWFpbGZsb3d8eyJFbXB0eU1hcGkiOnRydWUsIlYiOiIwLjAuMDAwMCIsIlAiOiJXaW4zMiIsIkFOIjoiTWFpbCIsIldUIjoyfQ%3D%3D%7C0%7C%7C%7C&sdata=3NH7zgOS%2BapQ7j%2F8izl%2F3xtZRnN5RnHfAJGsfB3%2Favg%3D&reserved=0)
- クラスター間ライブ マイグレーション:
SCVMM では、クラスター間で仮想マシンをライブ マイグレーションすることがサポートされております。
[VMM ファブリックでライブ マイグレーションを実行する | Microsoft Learn](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Flearn.microsoft.com%2Fja-jp%2Fsystem-center%2Fvmm%2Fmigrate-live%3Fview%3Dsc-vmm-2025%26tabs%3DSharedStorage&data=05%7C02%7CAtsushi.Hoshina%40microsoft.com%7Cdd62472f8425493f517a08dd62bc113a%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C638775282115848558%7CUnknown%7CTWFpbGZsb3d8eyJFbXB0eU1hcGkiOnRydWUsIlYiOiIwLjAuMDAwMCIsIlAiOiJXaW4zMiIsIkFOIjoiTWFpbCIsIldUIjoyfQ%3D%3D%7C0%7C%7C%7C&sdata=%2BJP5E56qGXcKwpkxz3Z0mxKv8cKyC2CLua6%2BZb0O5R0%3D&reserved=0)
操作自体は SCVMM を使用することなく実施いただけますが、必要な操作が SCVMM を使用した場合は簡略化されているのが特徴です。
[SCVMM のクラスター間ライブ マイグレーションの動作仕様、および仮想マシン移動時のエラーの対処方法 | Japan System Center Support Blog](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fjpsystemcenter.github.io%2Fblog%2FSCVMM%2FSCVMM_LiveMigration_and_Errors%2F&data=05%7C02%7CAtsushi.Hoshina%40microsoft.com%7Cdd62472f8425493f517a08dd62bc113a%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C638775282115855145%7CUnknown%7CTWFpbGZsb3d8eyJFbXB0eU1hcGkiOnRydWUsIlYiOiIwLjAuMDAwMCIsIlAiOiJXaW4zMiIsIkFOIjoiTWFpbCIsIldUIjoyfQ%3D%3D%7C0%7C%7C%7C&sdata=YAX%2F21LO5qJlttcDNjqt4Nl0%2Fgwd6zTueBTOi5RFP9Q%3D&reserved=0)

# Hyper-V マネージャー / WAC の機能
Hyper-V マネージャーや WAC を使用することで、Hyper-V の各種操作が可能となります。
具体的には以下が該当いたします。
- 仮想マシンの作成
- 仮想ディスクの作成
- 仮想マシンのインポート/エクスポート
- チェックポイントの作成
- 仮想マシンの開始/停止
- 仮想ディスクの拡張/縮小
- 仮想スイッチの作成/削除
これらの設定は SCVMM でも行うことが可能です。

なお、WAC は厳密には Hyper-V に限らず、Windows に関する各種設定事項や Azure との機能連携も可能となります。
本件は SCVMM の観点からの機能紹介となりますので、Hyper-V に関連しない WAC の操作事項に関して詳細は割愛できればと思います。
WAC で利用可能な Hyper-V 以外の機能のご紹介に留められればと存じますが、具体的には下記操作を WAC によって実現可能です。
- Azure Backup
- Azure Kubernetes サービス
- Azure Monitor
- クラスターの管理 (クラスター マネージャー相当)
- PowerShel の実行
- イベント ログの確認
- サービスの状態の確認
- タスク実行状況 (タスク スケジューラ相当) の確認
- デバイス状態 (デバイス マネージャ相当) の確認
- パケット キャプチャの採取
- パフォーマンス (パフォーマンス モニター相当) の確認
- 役割と機能の追加/削除
- リモート デスクトップ
- 記憶域レプリカの管理
- レジストリの操作
- ユーザーの追加/削除
- Windows Update の適用

# SCVMM でのみ利用可能な機能
  
Hyper-V や WAC では実現できず、SCVMM を使用することで実現可能な機能としては下記が挙げられます。
- 各種 Hyper-V ホストやクラスター、VMWare ESXi ホストやクラスターの管理、およびそれらの上で動作する仮想マシンの一元管理
- 仮想化ホストや仮想マシンに関するネットワーク設定の一元管理
- ストレージ機器の一元管理と仮想化ホスト / 仮想マシンへのストレージ機器の割り当て
- プライベート クラウドの運用
- インフラストラクチャ サーバーの管理による、より SCVMM による一元管理を実現できる仮想化ホスト / 仮想マシン管理
- SCVMM を経由した仮想化ホストの追加、プロビジョニング、管理
- VM テンプレートの利用することによる同一設定仮想マシンのデプロイの容易化
- SCVMM と登録仮想化ホスト / 仮想マシンのAzure 管理
- VMWare 仮想マシンを Hyper-V 仮想マシンに変換
- クラスター間ライブ マイグレーションの容易化

Hyper-V マネージャーや WAC が提供する仮想マシンの管理操作に対して、SCVMM はそれを大きく拡張することが可能なアプリケーションとしてご認識いただければ問題ございません。


---

SCVMM の概要紹介は以上となります。
これらの内容をレビューいただき、お客様の要件上 SCVMM の運用が適切かご判断いただく材料としてお役立てください。
最後までご覧いただきありがとうございました。