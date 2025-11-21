---
layout: default
title: Home Lab & Portfolio
---

# 中村雄介 – SRE / インフラエンジニア志望

電気系大学院修士（2025年卒）として制御設計職を経験した後、  
体調療養期間中に **3ノード Proxmox VE クラスタ** を中心とした自宅インフラ環境を構築・運用してきました。

現在は、**SRE / クラウドインフラ / バックエンドエンジニア** を主な志望として就職活動中です。

---

## プロフィール

- 年齢: 26歳
- 専攻: 電気系大学院 修士（2025年卒）
- 元職種: 車載空調系 制御設計エンジニア
- 強み:
  - Linux / 仮想化 / ネットワーク / ストレージの横断的な理解
  - 自宅での 3ノード Proxmox クラスタ構築・運用経験
  - OPNsense, TrueNAS, Caddy, Step-CA などインフラ要素技術の実践経験
  - C# / TypeScript / Node.js によるツール・サービス開発経験
- 希望ポジション:
  - SRE / プラットフォームエンジニア
  - クラウドインフラエンジニア
  - バックエンドエンジニア（インフラに強い開発者）

---

## スキルセット概要

### インフラ / プラットフォーム

- Linux（主に Debian / Ubuntu）: 利用歴 約3年
- Proxmox VE:
  - 3ノードクラスタ構築
  - VM / LXC, HA, ストレージプール
  - GPUパススルー (NVIDIA / Intel / AMD)
- ネットワーク:
  - OPNsense (25.7)
  - VXLAN / SDN / 複数セグメント設計
  - WireGuard VPN / DNS over TLS / DNS上書き設定
- ストレージ:
  - TrueNAS SCALE (25.04)
  - ZFS / NFS / Replication / Scrub / SMART監視
- セキュリティ / 証明書:
  - Step-CA による内部 CA 構築
  - ACME HTTP-01 認証で Proxmox / OPNsense の証明書発行
  - Caddy + Cloudflare DNS によるワイルドカード証明書運用

### 開発 / 自動化

- 言語:
  - C#（CLI / GUI (WinForms / Avalonia) アプリケーション）
  - TypeScript / Node.js（Bot / 小規模Web）
- DB:
  - MariaDB
- 自動化:
  - Ansible（環境構築の自動化を準備中）
- その他:
  - Discordチャット読み上げBot（辞書機能・DB利用）
  - 研究室向けデータ整理アプリ（作業時間を約90%削減）

---

## Home Lab 概要

療養期間中の学習と実践の場として、  
**自宅に Proxmox VE を中心とした小規模な「プライベートクラウド」環境**を構築しました。

### クラスタ構成

- Proxmox VE 8.x クラスタ
  - ノード数: 3
  - VM/LXC: 17台程度
  - ストレージ: ローカルNVMe + TrueNAS (NFS)
- 代表的なノード構成（例）:
  - Ryzen 9 / 7 クラス CPU
  - メモリ 64GB クラス
  - NVMe SSD システムディスク
  - dGPU: RTX 3050 / Intel Arc A310 / Radeon HD 8450

### ネットワークセグメント（VXLAN）

- `manage` : 10.0.10.0/24 – 管理系
- `remote` : 10.0.11.0/24 – リモート/VPN系
- `nas`    : 10.0.20.0/24 – ストレージ系
- `service`: 10.0.30.0/24 – 各種サービス
- `family` : 10.0.40.0/24 – 家族向け
- `public` : 10.0.50.0/24 – DMZ / 公開系

Proxmox の SDN機能と VXLAN を利用し、  
**物理構成に依存しない柔軟なセグメント構成**を実現しています。

---

## ネットワーク & セキュリティ

### OPNsense HA 構成

- OPNsense 25.7 を 2台のVMで構築
- CARPによる仮想IP:
  - WAN / LAN / 各 VXLAN セグメントに仮想IPを設定
- pfSync によるステート同期
- DNS / DHCP:
  - Unbound + Dnsmasq による名前解決・DHCP提供
  - 内部ドメイン: `int.sirosuzu.net`（例）

### ゾーン設計例

- TRUST_ZONE（管理系）
- STORAGE_ZONE（NAS）
- SERVICE_ZONE（サービス群）
- FAMILY_ZONE（家庭用）
- DMZ_ZONE（公開サービス）
- REMOTE_ZONE（VPN接続）

各ゾーン間での通信は、  
**最小限の許可ルールとエイリアス管理**により制御しています。

---

## ストレージ構成（TrueNAS）

### メイン TrueNAS

- TrueNAS SCALE 25.04
- ZFS プール:
  - 4TB HDD ×2 のミラー構成
- 主なデータセット:
  - `nextcloud` : Nextcloud データ
  - `media/immich_photo` : Immich 写真・動画
  - `nfs_share` : 汎用NFS
  - `proxmox/backup` : Proxmox VMバックアップ
  - `proxmox/vm` : HA 構成 VM用

### サブ TrueNAS

- 別ノード上の TrueNAS VM によるバックアップ先
- ZFS レプリケーションにより
  - Nextcloud / メディア / NFS / Proxmox 関連データを定期同期
- スケジュール:
  - 6時間ごと / 1日ごとなど、データ種別に応じて設定

---

## 主要サービス

### 1. Nextcloud

- 用途: 個人用ファイル同期・グループウェア
- ストレージ: TrueNAS NFS
- パフォーマンス:
  - Webサーバー: Caddy経由でリバースプロキシ
  - 大容量アップロードに対応するため `request_body` 調整

### 2. Immich

- 用途: 写真・動画バックアップサービス（セルフホスト版 Google Photos 的な位置付け）
- 構成:
  - Docker Compose 上で稼働
  - TrueNAS NFS 上の専用データセットをマウント
- チューニング:
  - メモリ使用量やキャッシュ挙動を監視しつつ調整

### 3. Forgejo

- 用途: Git ホスティング
- 今後の計画:
  - インフラ構成やAnsible Playbookを公開予定
  - IssueとWIPを使ったタスク管理

### 4. Pulse

- 用途: Proxmox ノード・ゲストの監視
- Proxmoxとの連携:
  - 専用コンテナで稼働
  - ノードごとの負荷・温度・メモリ使用量などを監視
- 通知:
  - Discord Webhook によるアラート

---

## 証明書・TLS管理（Step-CA / Caddy）

### Step-CA（内部CA）

- 目的: 内部サービス向けに信頼された証明書を配布
- 特徴:
  - ACME 対応 (`step-ca` の ACMEエンドポイント)
  - HTTP-01チャレンジで Proxmox / OPNsense の証明書を自動更新
- 運用:
  - 有効期限・最大期間を調整しつつ、更新頻度と運用負荷のバランスを最適化

### Caddy（リバースプロキシ）

- 用途:
  - 内部 / 外部サービスのリバースプロキシ
  - Cloudflare DNS + Let’s Encrypt による証明書自動取得
- 特徴:
  - Caddyfile による宣言的設定
  - Cloudflareプラグインを利用したDNS-01チャレンジ
  - Step-CAを利用した内部向けHTTPS化も一部で実施

---

## 自動化（Ansible – 作成中）

現在、Home Lab の一部構成を **Ansible で自動構築できるように整理中**です。

### 想定しているPlaybook例

- Ubuntu VM初期設定
  - ユーザー・SSH鍵・基本パッケージ
- Docker環境構築
- Immich / Nextcloud / 監視系の自動デプロイ

今後、リポジトリとして公開予定です。

---

## 開発実績（概要）

### C# アプリケーション

- 研究室向けデータ整理アプリ
  - 膨大な実験データの整理・集計を自動化
  - 作業時間を**約90%削減**
  - GUI (WinForms / Avalonia) による操作性を重視
- Discordチャット読み上げBot
  - C# + Discord API
  - 辞書機能付き（DB利用）
  - 一部 TypeScript / Node.js 版も作成

### Web / Backend

- TypeScript / Node.js でのBot・簡易Web
- 今後は
  - Go または TypeScript による小型API
  - Prometheus / OpenTelemetry との連携
 などを実装予定です。

---

## 今後の展望

- IaC:
  - Ansible / Terraform による構成管理のコード化
- クラウド:
  - Proxmoxでの経験を活かし、AWS / GCP / Azure への展開を学習
- SRE:
  - 監視 / ログ / トレーシング基盤の構築（Prometheus, Grafana, Lokiなど）
  - エラーバジェット / SLO 設計も学んでいきたいと考えています。
- 開発:
  - バックエンド開発とインフラの両方を理解したエンジニアとして成長したいです。

---

## 連絡先

- GitHub: [https://github.com/<your-username>](https://github.com/<your-username>)
- （必要に応じて）メールアドレスやX(Twitter)など

就職活動中のため、  
**SRE / インフラ / バックエンドポジションに興味をお持ちいただけた企業様がいらっしゃいましたら、ぜひご連絡ください。**
