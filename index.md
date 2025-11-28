---
layout: default
title: Home Lab & SRE Portfolio
---

# 名前 – SRE / インフラエンジニア志望

アルファベット名  
（2025 求職中・フレックス/ハイブリッド希望）

療養期間中に技術スキルの維持と向上、そして実用目的のために  
**3ノード Proxmox VE クラスタを中核としたプライベートクラウド環境**を構築し、  
OS / ネットワーク / ストレージ / セキュリティ / 監視 / 自動化 を横断的に学習・運用してきました。

---

# 👤 プロフィール

- 年齢：26歳  
- 学歴：電気系修士（2024年修了）  
- 元職種：車載空調制御（制御設計職）  
- 志望：**SRE / インフラ / バックエンドの3軸で応募中**  
- 働き方：**ハイブリッド勤務・フレックス希望**（睡眠相後退症候群の医師診断による）

---

# 🧠 保有スキル（概要）

### インフラ・仮想化
- Proxmox VE（クラスタ / HA / GPU Passthrough）
- KVM / LXC / QEMU  
- Linux（Ubuntu / Debian）3年以上運用

### ネットワーク
- OPNsense（25.7）  
- Proxmox SDN / VXLAN  
- DNS（Unbound）/ DHCP（Dnsmasq）  
- WireGuard / DNS over TLS（Quad9）

### ストレージ
- TrueNAS SCALE（25.04）  
- ZFS（ミラー構成）  
- NFS / レプリケーション / Scrub / SMART監視

### セキュリティ / TLS
- Step-CA（ACME運用）  
- Caddy（Reverse Proxy + DNS-01）  
- Cloudflare DNS管理

### 自動化 / IaC
- Ansible（Docker / VMプロビジョニング）  
- Cloud-init  
- Git / Forgejo

### 開発
- C#: GUI（Forms, Avalonia）/ CLI  
- TypeScript / Node.js（Discord Bot / API）  
- MariaDB

---

# 🏗️ Home Lab 概要（3ノード Proxmox クラスタ）

療養期間の学習基盤として、  
**小規模ながら商用レベルに近いプライベートクラウド環境**を構築しました。

```
┌──────────────────────────────────────┐
│ Proxmox VE (3 nodes / 19VM)          │
│ ├─ VXLAN (6 segments)                │
│ ├─ OPNsense HA                       │
│ ├─ TrueNAS SCALE                     │
│ ├─ Reverse Proxy (Caddy)             │
│ ├─ Step-CA                           │
│ └─ 各種サービス（Nextcloud, Immich 等） │
└──────────────────────────────────────┘
```


### ノード構成（例）

- ノードA：Ryzen 9 3900X / 48GB RAM  
- ノードB：Ryzen 7 3700X / 64GB RAM  
- ノードC：Core i5-10400 / 32GB RAM  
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
- ゾーンベースの**ゼロトラスト設計**  
- DNS上書きは Unbound に集約  
- WireGuard によるリモートアクセス

---

# 💽 ストレージ構成（TrueNAS）

### メインNAS

- TrueNAS SCALE 25.04（VM）  
- ZFS（4TB ×2 のミラー）  
- Nextcloud / Immich / Proxmox Backup をDataset単位で分離

### サブNAS（バックアップ）

- 別ノード上の TrueNAS VM  
- ZFSレプリケーションで差分同期  
- daily / weekly / monthly の世代管理

---

# 🧱 サービス一覧（主要）

| サービス | 概要 | 技術ポイント |
|---|---|---|
| Nextcloud | ファイル同期 | NFS + Caddy + TLS |
| Immich | 写真バックアップ | Docker Compose + NFS + Caddy |
| Forgejo | Gitホスティング | Reverse Proxy + TLS |
| Pulse | Proxmox監視 | ノード/VMの可視化 |
| Caddy | Reverse Proxy | Cloudflare DNS-01 |
| Step-CA | 内部ACME | Proxmox/OPNsense証明書更新 |

---

# 🔥 障害対応・トラブルシューティング例

### 1. Proxmox ノードが正常にシャットダウンできない

**原因**：IOMMU設定とBIOS仕様の相互作用  
**対策**：  
- kernel command line 調整  
- BIOS設定再構成  
→ **安定した停止処理を実現**

### 2. Proxmox → TrueNAS（NFS）バックアップの不整合

**原因**：コンテナのストレージ移動時にコンテナ設定が破損  
**対策**：コンテナ設定の整合性修正

### 3. OPNsense ACME が正常動作しない

**原因**：HTTP-01 経路問題とプラグイン仕様  
**対策**：  
- Step-CA の内部CAで証明書管理  
- 有効期間は長期化し手動更新（3ヶ月）  
→ 信頼性優先の運用に切り替え

### 4. Immich のDockerが高メモリ使用

**原因**：キャッシュ挙動  
**対策**：Pulseで監視し、しきい値調整で安定化

### 5. Immich Androidアプリが内部CAを信頼しない

**原因**：Android側の仕様で追加CAを信頼不可  
**対策**：Let's Encryptへ切り替え  
→ **アプリ互換性とセキュリティを両立**

---

# ⚙️ 自動化（Ansible）

現在、自宅環境のコード化を進めています。
Cloud-initでVMを作成し、AnsibleでVM内の設定を自動化しています。

### Playbook例

- Ubuntu 初期セットアップ  
- Docker CE インストール  
- Immich 自動デプロイ  
- NFS マウント管理

➡️ https://github.com/yusuke-524/ansible-immich

---

# 🚀 今後の学習・キャリア方向（固定していません）

### ✔ SRE（インフラ寄り）
- 監視 / SLO / 自動化  
- Kubernetes / Loki / Grafana 等

### ✔ クラウドインフラ
- AWS でのPoC  
- Terraform による IaC

### ✔ バックエンド
- Go / TypeScript で軽量API作成  
- Docker化 & Caddy公開

---

# 📬 連絡先

- GitHub: https://github.com/<your-username>  
- （任意）メール：<your-email>
