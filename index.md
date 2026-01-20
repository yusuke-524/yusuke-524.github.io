---
layout: default
title: 自宅サーバーポートフォリオ
---

# 中村雄介

Yusuke Nakamura  
（2026年 求職中・フレックス/ハイブリッド希望）

療養期間中に技術スキルの維持と向上、そして実用目的のために  
**3ノード Proxmox VE クラスタを中核としたプライベートクラウド環境**を構築し、  
OS / ネットワーク / ストレージ / セキュリティ / 監視 / 自動化 を横断的に学習・運用してきました。

---

# 👤 プロフィール
 
- 学歴：電気系修士（2024年修了）  
- 元職種：車載空調制御（制御設計職）  
- 志望：**主にインフラ / バックエンドの2軸で応募中**  
- 働き方：**ハイブリッド/リモート勤務・フレックス希望**

---

# 🧠 保有スキル（概要）

### インフラ・仮想化
- Proxmox VE 8.x（クラスタ / HA / GPU Passthrough / IOMMU）
- KVM / LXC / QEMU  
- Linux（Ubuntu / Debian）3年以上の自宅サーバー運用経験

### ネットワーク
- OPNsense 25.7（CARP + pfSync による HA構成）
- Proxmox SDN / VXLAN（6セグメント構成）
- DNS（Unbound）/ DHCP（Dnsmasq）  
- WireGuard VPN / DNS over TLS（Quad9）
- ゾーンベースファイアウォール設計

### ストレージ
- TrueNAS SCALE 25.04  
- ZFS（ミラー構成 / Scrub / SMART監視）  
- NFS 共有設定・マウント管理
- ZFS レプリケーション（日次/週次/月次の世代管理）

### セキュリティ / TLS
- Step-CA（プライベート認証局 / ACME運用）  
- Caddy（Reverse Proxy / DNS-01チャレンジ / Let's Encrypt）  
- Cloudflare DNS / API連携

### 自動化 / IaC
- Ansible（Docker / VM プロビジョニング実装済み）
- Cloud-init（VM テンプレート自動構築）
- Git / Forgejo（自宅Git環境）

### 開発経験（趣味 / 大学院研究用）
- C#: GUI（Windows Forms / Avalonia UI）/ CLI  
- TypeScript / Node.js（Discord Bot / REST API）  
- MariaDB / CouchDB

---

# 🏗️ Home Lab 概要（3ノード Proxmox クラスタ）

療養期間の学習基盤として、  
**小規模ながら商用レベルに近いプライベートクラウド環境**を構築しました。

**運用規模**
- VM/CT数：19台
- VXLANセグメント：6つ
- ストレージ：ZFSミラー 4TB × 2
- レプリケーション：日次/週次/月次管理

```
┌─────────────────────────────────────────┐
│ Proxmox VE (3 nodes / 19VM)             │
│ ├─ VXLAN (6 segments)                   │
│ ├─ OPNsense HA                          │
│ ├─ TrueNAS SCALE                        │
│ ├─ Reverse Proxy (Caddy)                │
│ ├─ Step-CA                              │
│ └─ 各種サービス（Nextcloud, Immich 等） │
└─────────────────────────────────────────┘
```


### 実ノード構成

| ノード  | CPU           | RAM  | GPU      | 用途                              |
| ------- | ------------- | ---- | -------- | --------------------------------- |
| pvecore | Ryzen 7 3700X | 64GB | GTX1650  | インフラ中核（NAS/OPNsense/監視） |
| pve1    | Ryzen 9 3900X | 48GB | Arc A310 | 汎用サービス                      |
| pve2    | Core i5-10400 | 32GB | -  | 汎用サービス + 開発/テスト環境    |

---

# 📊 運用実績

**構築・運用期間**
- 構築開始：2025年9月
- 現在の運用期間：4ヶ月
- 最長連続稼働：60日以上（計画メンテナンスを除く）

**安定性指標**
- 計画メンテナンス：月1回程度
- 自動復旧成功率：可能なサービスはHA構成により障害時も無停止運用を実現

---

# 🌐 ネットワーク・ストレージ構成

- **Proxmox SDN + VXLAN** による6セグメント論理分離
- **OPNsense HA**（CARP + pfSync）による完全冗長ルーター
- ゾーンベースの**ゼロトラスト設計**
- **TrueNAS SCALE** によるZFSミラー構成（4TB × 2）
- **ZFS レプリケーション**による日次/週次/月次バックアップ

→ [インフラ構成の詳細を見る](/details/infrastructure.html)

---

# 🧱 サービス一覧

**運用中の主要サービス（8種）**：OPNsense（HA構成）、TrueNAS、Step-CA、Caddy、Nextcloud、Immich、Forgejo、Pulse

→ [サービス一覧の詳細を見る](/details/services.html)

---

# 💾 バックアップ・DR戦略

**3層バックアップ構成**で堅牢なデータ保護を実現：

1. **ZFS スナップショット** — 重要コンテナは15分〜2時間毎
2. **Proxmox 標準バックアップ** — TrueNAS へ日次/週次/月次
3. **TrueNAS ZFS レプリケーション** — 別ノードへ自動同期

**災害復旧指標**：RPO 24時間以内 / RTO 2時間以内

→ [バックアップ・監視の詳細を見る](/details/operations.html)

---

# 🔍 監視・運用体制

- **Pulse**（Proxmox専用監視）によるクラスタ全体の可視化
- **Discord Webhook** による即時アラート通知
- サービス特性に応じた**カスタムしきい値**設定
- 定期メンテナンス：ZFS Scrub（月1回）、SMART監視（週1回）

---

# 🔒 セキュリティ対策

- **ゼロトラストアーキテクチャ**：6セグメント完全分離、原則通信不可
- **3層ファイアウォール**：OPNsense → Proxmox Datacenter → 各VM/CT
- **TLS 1.3 / SSH公開鍵認証 / 2FA** による認証・暗号化
- **Step-CA / Let's Encrypt** による証明書自動管理

→ [セキュリティ詳細を見る](/details/operations.html)

---

# 🔥 障害対応・トラブルシューティング例

### 1. Proxmox ノードが正常にシャットダウンできない

**症状**：特定ノードで `shutdown` がハング、強制停止が必要になる  
**原因**：IOMMU設定とBIOS不具合の相互作用でデバイス切離に失敗  
**対策**：  
- kernel command line で `amd_iommu=on iommu=pt` 設定  
- BIOS設定の見直し（BIOS不具合による代替設定）  
**結果**：安定した停止処理を実現、ノード再起動の信頼性向上

### 2. Proxmox → TrueNAS（NFS）バックアップ失敗

**症状**：特定コンテナのバックアップ時にハング、他のバックアップも影響を受ける  
**原因**：コンテナのストレージ移動時に `.conf` ファイルが破損（`[vzdump]` セクション残存）  
**対策**：設定ファイルの手動修正で `rootfs` 定義を正規化  
**学び**：コンテナ設定の整合性確認の重要性、バックアップ前の検証手順確立

### 3. Immich Docker が高メモリ使用でアラート

**症状**：メモリ使用率95%超え、でPulseからのアラート発生  
**原因**：Docker / snap のキャッシュ挙動によるメモリ消費  
**対策**：Pulse 監視でしきい値を調整（実際のメモリ圧迫ではないと判断）   
**学び**：メモリ使用率の表面的な数値だけでなく、実際の圧迫状況を見極める重要性

### 4. Immich Android アプリが内部 CA を信頼しない

**症状**：Step-CA で発行した証明書を Android アプリが拒否  
**原因**：Androidアプリ側の仕様で追加CAを信頼しない  
**対策**：Caddy 側の証明書を **Let's Encrypt** へ切り替え（DNS-01チャレンジ / Cloudflare API連携）  
**結果**：アプリ互換性とセキュリティを両立、外部からのアクセスも可能に

---

# 🎯 技術選定の理由

| 技術              | 採用理由                                                                                    | 検討した代替案                     |
| ----------------- | ------------------------------------------------------------------------------------------- | ---------------------------------- |
| **Proxmox VE**    | ・KVM/LXC 両対応<br>・WebUI が充実<br>・クラスタ・HA機能が標準<br>・オープンソース          |                 |
| **OPNsense**      | ・HA機能（CARP/pfSync）<br>・VXLAN 対応<br>・FreeBSD ベースの安定性<br>・WebUI の使いやすさ | pfSense, VyOS, OpenWrt             |
| **TrueNAS SCALE** | ・ZFS の信頼性<br>・レプリケーション機能<br>・NFS/SMB 統合管理<br>・WebUI                   |  FreeNAS Core       |
| **VXLAN**         | ・VLAN対応スイッチ不要<br>・既存ネットワーク上に構築可<br>・柔軟なセグメント設計            | 物理VLAN, Wireguard メッシュ       |
| **Caddy**         | ・自動TLS更新<br>・Caddyfile のシンプルさ<br>・DNS-01チャレンジ対応                         | Nginx, Traefik, HAProxy            |
| **Ansible**       | ・エージェントレス<br>・冪等性の保証<br>・YAML の可読性<br>・学習コスト低                   | Terraform            |
| **Step-CA**       | ・プライベートCA<br>・ACME対応<br>・軽量（コンテナで動作）                                  | 自己署名証明書, Let's Encrypt のみ |

---

# ⚙️ 自動化（Ansible + Cloud-init）

現在、自宅環境のコード化（IaC）を進めています。  
**Cloud-init でVM初期構築 → Ansible で設定自動化**のワークフローを確立。

### 実装内容

- **Ubuntu VM テンプレート作成**（Cloud-init）  
- **Ansible Playbook による自動デプロイ**  
  - OS初期設定（タイムゾーン / ロケール / パッケージ更新）  
  - Docker CE インストール  
  - Immich 自動デプロイ（docker-compose.yml / .env 配置）  
  - NFS マウント設定（/etc/fstab 管理）  
  - systemd サービス登録

### 成果

- VM構築時間を **手動1時間 → 自動10分** に短縮  
- 再現性の確保（何度でも同じ状態に復元可能）  
- ドキュメントのコード化（手順書が実行可能に）

➡️ GitHub リポジトリ: <https://github.com/yusuke-524/ansible-immich>

→ [ドキュメント管理・学習方法の詳細を見る](/details/services.html)

---

# 🚀 今後の学習・キャリア方向

### ✔ インフラエンジニア
- Kubernetes / Helm（自宅クラスタでの検証予定）  
- Prometheus + Grafana（現在 Pulse から移行検討中）  
- Terraform / Pulumi による IaC

### ✔ クラウドインフラ
- AWS での PoC（EC2 / S3 / RDS 等）  
- マルチクラウド構成の検証

### ✔ バックエンドエンジニア（副次的）
- Go / TypeScript による軽量 API 作成  
- コンテナ化 & Caddy 公開

---

# 📬 連絡先

- **GitHub**: <https://github.com/yusuke-524>  
- **ポートフォリオ**: <https://yusuke-524.github.io>