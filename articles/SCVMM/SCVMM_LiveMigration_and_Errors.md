﻿---
title: SCVMM のクラスター間ライブ マイグレーションの動作仕様、および仮想マシン移動時のエラーの対処方法
date: 2024-04-01 17:20:00
tags:
  - SCVMM
  - HowTo
  - Migration
disableDisclaimer: false
---

<!-- more -->
皆さんこんにちは、System Center サポートチームの 保科と申します。


今回は、System Center Virtual Machine Manager (SCVMM) の機能としてご利用いただける、仮想マシンのライブ マイグレーションやエラーへの対処方法についてご説明いたします。

<!-- more -->

### 本ページが対象とする SCVMM バージョン
SCVMM 2016
SCVMM 2019
SCVMM 2022

### 概要
SCVMM をご利用いただくメリットとして、多くの方が、クラスター間で仮想マシンを移動することが出来る点を挙げるかと思います。
少なくともフェールオーバー クラスター単体だと、SCVMM のように簡単に仮想マシンを移動 / ライブ マイグレーションすることが出来ません。
しかし、クラスター間の仮想マシン移動は、実はフェールオーバー クラスターや Hyper-V 単体においても実施いただけることをご存じでしょうか。
SCVMM でのクラスター間の仮想マシン移動は、この際必要となる一連の動作を 1 つの操作に纏めて実施しているにすぎません。
本日は、この機能がどのように実行されるかについてご紹介いたします。

また、仮想マシン移動時にエラーが発生した場合において、対処に際して見落としがちな設定等をご紹介いたします。
こちらは、今後弊社内にて確認した情報の増加に従って、本ブログの内容をアップデートとする予定です。

### SCVMM におけるクラスター間の仮想マシン移動やライブ マイグレーションの動作仕様
早速ですが、SCVMM において、仮想マシンはどうやってクラスター間で移動しているのか？
この動作の流れをご紹介いたします。
1. フェールオーバー クラスターに登録された、移動対象の仮想マシンの役割の削除
2. 移行先として指定された Hyper-V ホストに、移動対象の仮想マシンを移動、もしくはライブ マイグレーション
3. 仮想マシン移動先の Hyper-V ホストが所属するクラスターに、移動された仮想マシンの役割を登録

このような流れで、内部上処理が行われています。
フェールオーバー クラスター上で動作する仮想マシンは、クラスターの役割を登録するという形で、高可用性構成を行うことが可能です。
クラスター上に役割が登録された仮想マシンは、SCVMM 外においては、クラスター内のノード間でしか仮想マシンを移動することが出来ません。
そのため、SCVMM 外でクラスター間の仮想マシン移動を実施する場合、仮想マシンの役割をクラスターから一度削除する必要があります。
これを実施することで初めて別のクラスター内ノードに仮想マシンを移動することができます。
最後に仮想マシン移行先の Hyper-V ホストが所属するクラスターに、移動された仮想マシンの役割を登録することで、そのクラスター内においても高可用性構成を実現いただけます。
なお、仮想マシンの電源がオンの状態で移動操作を実施すると、電源がオンの状態で移動 (ライブ マイグレーション) が行われます。
こちらの動作のしくみや、Hyper-V マネージャーを使用した手順は、下記の弊社公開情報に記載されております。
[フェールオーバー クラスタ リングのないライブ マイグレーションを使用して仮想マシンを移動するには | Microsoft Learn](https://learn.microsoft.com/ja-jp/windows-server/virtualization/hyper-v/manage/use-live-migration-without-failover-clustering-to-move-a-virtual-machine)

SCVMM を使わない場合、上記の流れを手動で実施することで、クラスター間での仮想マシン移動が実現できます。
SCVMM は、この流れを 1 つの操作に纏めることで、ユーザー負担の軽減が実現できます。
逆に言えば、万が一 SCVMM をご利用いただけなくなった状況において、クラスター間の仮想マシン移動の実施が必要となった場合、上記手順を手動で実施いただくことで、SCVMM で実施するクラスター間仮想マシン移動と同じ操作が実現できます。

### 仮想マシンの移動に失敗するシナリオ
こちらは、弊社で情報が確認でき次第、随時内容をアップデートする予定です。

#### 仮想マシン移動失敗シナリオその 1: Hyper-V ホストの Credential Guard が有効になっている
Windows OS には Credential Guard という機能が存在します。
こちらの詳細は、下記の弊社公開情報に記載されております。
[Credential Guard のしくみ - Windows Security | Microsoft Learn](https://learn.microsoft.com/ja-jp/windows/security/identity-protection/credential-guard/how-it-works)
お客様の環境によってはこちらの機能が有効化されている可能性がありますが、仮想マシン移動時においては問題となる場合がございます。
その理由として、この機能が有効化されていると、仮想マシン移動時に使用される認証方式の 1 つである CredSSP が利用できなくなるように制限がかかるためです。
Kerberos 認証に対しては制限がかからないため、こちらの設定をご利用のお客様においては、以下のいずれかの実施が必要となります。
- Credential Guard を無効化する。
- 仮想マシン移動時の認証方式を CredSSP から Kerberos に変更する。

Credential Guard の有効化状況は、下記弊社公開情報の "Credential Guard が有効になっているかどうかを確認する" 項以下に記載された手順から確認いただけます。
[Credential Guard の構成 - Windows Security | Microsoft Learn](https://learn.microsoft.com/ja-jp/windows/security/identity-protection/credential-guard/configure?tabs=intune)
まずはこちらの手順で、Credential Guard が有効化されているかを確認いただく必要がございます。

お客様によっては、Credential Guard は、目的があって有効化されている状況が多いかと思います。
その場合は無効化が難しいと思うので、仮想マシン移動のシナリオがどうしても必要という状況であれば、認証方式を Kerberos に切り替える以外に対処方法はございません。
幸いこの操作は SCVMM 側において簡単に実施いただけますので、手順をご紹介いたします。
1. SCVMM サーバーに管理者権限でログインし、SCVMM コンソールを開きます。
2. SCVMM コンソール画面左下ペインの [ファブリック] をクリックします。
3. 画面左ペインの [サーバー] -> [すべてのホスト] をクリックします。
4. ライブマイグレーション移行元、および移行先のホストを右クリックし、[プロパティ] を開きます。
(クラスター間で仮想マシンのライブマイグレーションを実施される場合、各種クラスターに配置されたノード全台に対して設定を実施いただくことを推奨いたします。)
5. 表示されたプロパティ ウィンドウ左の [移行の設定] をクリックします。
6. [認証プロトコル] 項内の "Kerberos を使用する" チェックを入力し、[OK] をクリックして設定を保存します。

手順は以上となります。
同一ドメイン環境の仮想マシン移動であれば Kerberos 認証をご利用いただけますので、Kerberos 認証をご利用いただく設定が一般的となります。

#### 仮想マシン移動失敗シナリオその 2: 構成バージョンのサポート
Hyper-V 上に作成された仮想マシンは、いずれも例外なく構成バージョンという内容が構成されます。
この構成バージョンは OS 毎に既定で仮想マシン作成時に展開されるバージョンが決められております。
既定で展開される構成バージョンとは異なるバージョンで仮想マシンを展開いただく場合、コマンド経由で仮想マシンを作成いただく必要がございます。

今回の問題としては、仮想マシンの構成バージョンによっては、その構成バージョンが移行先ではサポートされていないバージョンに該当し、結果移行が失敗する可能性があるということです。
例えば以下の Hyper-V サーバーの構成を考えます。
- Windows Server 2016:
サポートされる最も新しい構成バージョンは 8.0 で、仮想マシン展開時の既定のバージョンもこちらになります。
構成バージョン 9.0 はサポートされません。
- Windows Server 2019:
サポートされる最も新しい構成バージョンは 9.0 で、仮想マシン展開時の既定のバージョンもこちらになります。
構成バージョン 8.0 はサポートされます。

この際、いずれも既定の構成バージョンの設定で、Windows Server 2019 Hyper-V 上で仮想マシンを作成したとします。
この際展開される仮想マシンは構成バージョンが 9.0 となります。
この仮想マシンを Windows Server 2016 Hyper-V に移行しようとすると、移行先がサポートしない構成バージョンであるとして、移行に失敗します。
例えば SCVMM コンソール上でこの移行操作を実施すると、以下エラーが発生して操作が失敗します。
![](SCVMM_LiveMigration_and_Errors/SCVMM_VMVersion_MigrationError.png)

逆のパターンとして、Windows Server 2016 で作成された仮想マシンを Windows Server 2019 に移行するシナリオを考えます。
Windows Server 2016 Hyper-V で作成される仮想マシンは、既定で構成バージョンが 8.0 となります。
Windows Server 2019 Hyper-V では構成バージョン 8.0 の仮想マシンはサポートされております。
そのため、構成バージョンが 8.0 で動作している Windows Server 2016 Hyper-V の仮想マシンは、問題なく Windows Server 2019 Hyper-V へ移行できます。

構成バージョンのアップグレードは可能ですが、ダウングレードは実施出来ません。
そのため、構成バージョン 9.0 の仮想マシンを Windows Server 2016 Hyper-V に移行することは出来ません。
移行の際は、構成バージョンについても十分にご留意いただいた上で操作を実施いただく必要があるとご認識をいただけると幸いです。

仮想マシンの構成バージョンに関する情報は、以下の弊社公開情報に詳細が纏められております。
[Hyper-V の仮想マシンのバージョンをアップグレードする (Windows または Windows Server) | Microsoft Learn](https://learn.microsoft.com/ja-jp/windows-server/virtualization/hyper-v/deploy/upgrade-virtual-machine-version-in-hyper-v-on-windows-or-windows-server)
OS による構成バージョンのサポートや、各構成バージョンで利用可能な機能といった、大変有用な情報が纏められております。