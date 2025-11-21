---
layout: default
title: Home Lab & SRE Portfolio
---

# <名前> – SRE / インフラエンジニア志望  
<アルファベット名>
（2025 求職中・フレックス/ハイブリッド希望）

私は療養期間中に、技術スキル習得と向上のため及び実用を目的に、  
**3ノード Proxmox VE クラスタを中心としたプライベートクラウド環境**を構築し、  
OS / ネットワーク / ストレージ / セキュリティ / 監視 / 自動化 を横断的に学習してきました。

---

# 👤 プロフィール

- 年齢：26歳  
- 学歴：電気系修士（2024年修了）  
- 元職種：車載制御（空調制御設計）  
- 求職状況：SRE / インフラ / バックエンドの3軸で応募中  
- 働き方：**ハイブリッド勤務・フレックスを希望**（睡眠相後退症候群への医師診断に基づく）

---

# 🧠 保有スキル（概要）

### インフラ・仮想化
- Proxmox VE（クラスタ / HA / GPU Passthrough）
- LXC / KVM / QEMU
- Linux（Ubuntu / Debian）運用歴 3年

### ネットワーク
- OPNsense（25.7）
- VXLAN / Proxmox SDN
- DNS（Unbound）/ DHCP（Dnsmasq）
- WireGuard / DoT（Quad9）

### ストレージ
- TrueNAS SCALE（25.04）
- ZFS / Scrub / SMART
- NFS / レプリケーション

### セキュリティ / TLS
- Step-CA（ACME）
- Caddy（Reverse Proxy + DNS-01）
- Cloudflare DNS

### 自動化 / IaC
- Ansible（Docker / VMプロビジョニング）
- Cloud-init
- Git / Forgejo

### 開発
- C#: GUI（Forms, Avalonia）/ CLI
- TypeScript / Node.js（Discord Bot / API）
- DB（MariaDB）

---

# 🏗️ Home Lab 概要（3ノード Proxmox クラスタ）

療養中の学習の中心として、  
**自宅に小規模ながら本格的な「プライベートクラウド」環境**を構築しました。

```
┌──────────────────────────────────────┐
│ Proxmox VE (3 nodes / 19VM)          │
│ ├─ VXLAN (7 segments)                │
│ ├─ OPNsense HA                       │
│ ├─ TrueNAS SCALE                     │
│ ├─ Reverse Proxy (Caddy)             │
│ ├─ Step-CA                           │
│ └─ 各種サービス（Nextcloud, Immich 等） │
└──────────────────────────────────────┘
```

### ノード構成（例）
- ノードA：Ryzen 7 3700X / 64GB RAM  
- ノードB：Core i5-10400 / 32GB RAM  
- ノードC：Ryzen 9 3900X / 48GB RAM  
- GPU：RTX3050 / Intel Arc A310 / GTX1650

---

# 🌐 ネットワーク構成（VXLAN + OPNsense HA）

### セグメント
- manage: `10.0.10.0/24`
- remote: `10.0.11.0/24`
- nas: `10.0.20.0/24`
- service: `10.0.30.0/24`
- family: `10.0.40.0/24`
- public: `10.0.50.0/24`

### ルータ（OPNsense HA）
- CARP + pfSync による冗長構成  
- ルールは **ゼロトラストを意識したゾーン間通信制御**  
- DNS Unboundで上書き設定
- VPN（WireGuard）でリモートアクセス

---

# 💽 ストレージ構成（TrueNAS）

### メインNAS
- TrueNAS SCALE 25.04（VM）
- ZFS（4TB ×2 mirror）  
- Nextcloud / Immich / Proxmoxバックアップを専用Datasetで分離

### サブNAS
- 別ノードに TrueNAS VM  
- レプリケーションで差分同期  
- Proxmoxバックアップの世代保持（daily/weekly/monthly）

---

# 🧱 サービス一覧（主要）

| サービス | 概要 | 技術ポイント |
|---|---|---|
| Nextcloud | ファイル同期 | NFS + Caddy + TLS |
| Immich | 写真バックアップ | Docker Compose + NFS + Caddy |
| Forgejo | Gitホスティング | Caddy |
| Pulse | 監視 | Proxmox 連携 |
| Caddy | Reverse Proxy | Cloudflare DNS-01 |
| Step-CA | 内部ACME | Proxmoxなど管理系サービス証明書 |

---

# 🔥 障害対応・トラブルシューティング例

実際に遭遇した問題と対策を抜粋しています。

### 1. Proxmox ノードが shutdown 時にハング  
**原因**：IOMMU設定＋BIOS仕様  
**対処**：カーネルパラメータ調整、GPU出力先検証、BIOS再構成

### 2. Proxmox→TrueNAS（NFS）でバックアップが失敗  
**原因**：NFSv3の属性扱い・スナップショット不整合  
**対策**：コンテナ設定ファイルの点検

### 3. OPNsense ACMEが正常動作しない  
**原因**：HTTPチャレンジ到達不可＋設定仕様  
**対策**：Step-CAによる手動ACME構築

### 4. Immich のDockerコンテナが高メモリ消費  
**原因**：キャッシュ挙動  
**対策**：監視（Pulse）でトレースし、しきい値調整

---

# ⚙️ 自動化（Ansible）

現在、自宅環境の構成を **Ansibleでコード化** しています。

### 提供予定のPlaybook例
- Ubuntu初期設定  
- Docker CE導入  
- Immich自動デプロイ  
- NFSマウント管理  

➡️ リポジトリ公開予定（`ansible-home-lab`）

---

# 🚀 今後の学習・キャリア方向（まだ固定していません）

私は現在、以下の3分野を探索中です：

### ✔ SRE（インフラ寄り）
- 監視・SLO・自動化・Kubernetes  
- ログ基盤（Loki/Grafana）を導入予定  

### ✔ クラウドインフラ
- AWSなどを用いて「HomeLabをクラウドに写す」活動予定  
- IaC（Terraform）にも挑戦したい  

### ✔ バックエンド
- Go/TypeScriptでAPIを書く練習  
- 小さなAPI＋Docker化 → 外部用Caddy経由で公開予定

---

# 📬 連絡先
- GitHub: https://github.com/<your-username>  
- （任意）メール：<your-email>  
- （任意）ポートフォリオのPDF版

