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
- 構築開始：2024年9月
- 現在の運用期間：4ヶ月
- 最長連続稼働：60日以上（計画メンテナンスを除く）

**安定性指標**
- 計画メンテナンス：月1回程度
- 自動復旧成功率：可能なサービスはHA構成により障害時も無停止運用を実現

---

# 🌐 ネットワーク構成（VXLAN + OPNsense HA）

### セグメント設計

Proxmox SDN + VXLAN による論理分離（MTU 1450）

| セグメント | CIDR         | 用途                     | 備考            |
| ---------- | ------------ | ------------------------ | --------------- |
| manage     | 10.0.10.0/24 | サーバー管理・証明書配布 | Step-CA配置     |
| remote     | 10.0.11.0/24 | リモートアクセス         | WireGuard VPN   |
| nas        | 10.0.20.0/24 | ストレージ               | TrueNAS / NFS   |
| service    | 10.0.30.0/24 | 個人サービス             | Caddy Proxy配下 |
| family     | 10.0.40.0/24 | 家族向けサービス         | AdGuard Home    |
| public     | 10.0.50.0/24 | 外部公開                 | Minecraft等     |

### ルーター（OPNsense HA）

- **CARP + pfSync** による完全冗長構成  
- 仮想IP（VIP）によるフェイルオーバー  
- ゾーンベースの**ゼロトラスト設計**  
  - セグメント間は原則通信不可  
  - 必要な通信のみポート/IP単位で許可  
- DNS上書き（Unbound）で内部名前解決  
- DNS over TLS（Quad9 上流）  
- WireGuard によるセキュアなリモートアクセス

---

# 💽 ストレージ構成（TrueNAS）

### メインNAS（VM on pvecore）

- TrueNAS SCALE 25.04  
- ZFS ミラー構成（東芝 N300 4TB × 2）  
- SATA コントローラー直接パススルー（IOMMU）  
- Dataset単位でサービス分離  
  - Nextcloud / Immich / Proxmox Backup Server 用  
- NFS 共有設定

### サブNAS（バックアップ VM on pve1）

- ZFS レプリケーションによる差分バックアップ  
- **日次 / 週次 / 月次の世代管理**  
- ノード障害時のデータ保全

---

# 🧱 サービス一覧（主要）

| サービス      | 概要             | 技術ポイント                                 | 配置           |
| ------------- | ---------------- | -------------------------------------------- | -------------- |
| **OPNsense**  | ルーター         | CARP/pfSync HA / VXLAN統合                   | pvecore + pve1 |
| **TrueNAS**   | NAS              | ZFS / レプリケーション / NFS                 | pvecore        |
| **Step-CA**   | プライベートCA   | ACME / HTTP-01                               | CT(manage)     |
| **Caddy**     | Reverse Proxy    | DNS-01 / Let's Encrypt / Cloudflare API      | CT(service)    |
| **Nextcloud** | ファイル同期     | NFS + Caddy + TLS                            | VM(nas)        |
| **Immich**    | 写真バックアップ | Docker Compose + NFS + 自動デプロイ(Ansible) | VM(nas)        |
| **Forgejo**   | Git ホスティング | Reverse Proxy + TLS                          | CT(service)    |
| **Pulse**     | Proxmox 監視     | クラスタ全体可視化 / Discord通知             | CT(manage)     |

---

# 💾 バックアップ・DR戦略

### バックアップ体系

**3層バックアップ構成**
1. **ZFS スナップショット**（VM/CT の HA 対応）
   - 重要コンテナ：15分〜2時間毎
   - ノード間レプリケーション（pvecore ⇄ pve1 ⇄ pve2）
2. **Proxmox 標準バックアップ**（TrueNAS へ）
3. **TrueNAS ZFS レプリケーション**（メインNAS → サブNAS）

### VM/CT バックアップスケジュール

| バックアップ種別 | 頻度           | 保持期間 | 対象                            | 保存先  |
| ---------------- | -------------- | -------- | ------------------------------- | ------- |
| **日次**         | 月〜金 AM2:00  | 3日      | 全VM/CT（TrueNAS/OPNsense除く） | TrueNAS |
| **日次（サブ）** | 月〜金 AM3:00  | 7日      | 全VM/CT（TrueNAS/OPNsense除く） | サブNAS |
| **週次**         | 土曜 AM2:00    | 2週間    | 全VM/CT（テスト環境除く）       | TrueNAS |
| **週次（サブ）** | 土曜 AM3:00    | 3週間    | 全VM/CT（テスト環境除く）       | サブNAS |
| **月次**         | 第1日曜 AM4:00 | 2ヶ月    | 本番環境のみ                    | TrueNAS |
| **月次（サブ）** | 第1日曜 AM5:00 | 3ヶ月    | 本番環境のみ                    | サブNAS |

**特徴**
- スナップショットモード（Proxmox 標準機能）
- ZSTD 圧縮による容量最適化
- 重複排除のため日次と週次をスケジュール調整

### TrueNAS レプリケーション

- **メインNAS → サブNAS** への自動同期
- 日次 / 週次 / 月次の3世代管理
- 異なるノード間でのデータ保全（ノード障害対策）

### 災害復旧指標

- **RPO（目標復旧時点）**：24時間以内（日次バックアップ）
- **RTO（目標復旧時間）**：2時間以内（手動リストア）
- **バックアップテスト**：月1回、ランダムなVM/CTでリストア検証

---

# 🔍 監視・運用体制

### 監視ツール：Pulse（Proxmox専用監視）

**監視項目**
- CPU / メモリ / ディスク / ネットワーク使用率
- VM/CT 稼働状況（起動/停止）
- ノード間クラスタ状態
- ストレージ容量・健全性

**アラート通知**
- Discord Webhook による即時通知
- VM/CT ごとにカスタマイズしたしきい値設定

**しきい値設定例**

| VM/CT        | CPU | メモリ | 理由                           |
| ------------ | --- | ------ | ------------------------------ |
| 標準サービス | 80% | 90%    | 通常の負荷監視                 |
| TrueNAS      | -   | 95%    | ZFS がメモリを積極利用するため |
| Nextcloud    | -   | 95%    | snap のキャッシュ挙動を考慮    |
| Immich       | -   | 95%    | Docker のキャッシュ挙動を考慮  |

**定期メンテナンス**
- ZFS Scrub：月1回（データ整合性チェック）
- SMART監視：週1回（ディスク健全性確認）
- セキュリティアップデート：週1回
- バックアップリストアテスト：月1回

---

# 🔒 セキュリティ対策

### ネットワークセキュリティ

**ゼロトラストアーキテクチャ**
- 6つのセグメント（VXLAN）による完全分離
- セグメント間は原則通信不可
- 必要な通信のみポート/IP単位で許可

**ファイアウォール階層**
1. OPNsense：セグメント間ルーティング・ファイアウォール
2. Proxmox Datacenter：ノード間通信制御
3. 各VM/CT：個別のファイアウォール設定

### 認証・暗号化

- **TLS 1.3**：ほぼ全サービスでTLS通信を強制
- **SSH**：公開鍵認証のみ（パスワード認証無効化）
- **2FA**：ProxmoxVE管理画面 / TrueNAS などで二要素認証を有効化
- **VPN**：WireGuard による暗号化リモートアクセス

### 証明書管理

- **内部サービス**：Step-CA（プライベートCA）によるACME自動更新
- **外部互換性が必要なサービス**：Let's Encrypt（DNS-01チャレンジ）
- 証明書有効期限：30日（自動更新）

### アクセス制御

- 管理セグメント（10.0.10.0/24）は VPN 経由およびOPNsense LANからのみアクセス可
- Proxmox 管理画面は物理LAN（192.168.0.0/24）、VPN 経由およびOPNsense LANからのみアクセス可
- 外部公開サービスは専用セグメント（10.0.50.0/24）に隔離

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

---

# 📚 ドキュメント管理体制

### 構築ドキュメント（Obsidian）

**記録内容**
- 19台の VM/CT それぞれに専用ドキュメント
- 構築手順・設定ファイル・トラブルシューティングを詳細記録
- セグメント設計・ネットワーク構成図
- 変更履歴（Changelog）の記録

**ドキュメント構成例**
```
docs-internal/
├── サーバー全体構成(Ver2.5).md
├─VM/
│ ├── インフラ系VM/
│ │ ├── OPNsense.md
│ │ ├── TrueNAS.md
│ │ ├── Caddy.md
│ │ └── cert-manager(Step-CA).md
│ ├── サービス系VM/
│ │ ├── Immich.md
│ │ ├── Nextcloud.md
│ │ └── Forgejo.md
│ └── ...
└── cloud-init.md
```

### 設定ファイル管理（Forgejo / Git）

- Ansible Playbook
- Docker Compose ファイル
- システム設定ファイルのバージョン管理
- 変更履歴の追跡

### Obsidian 同期

- CouchDB による複数デバイス間の同期
- 自宅サーバー内で完結（外部サービス不使用）

---

# 📖 学習方法・リソース

- **公式ドキュメント精読**：Proxmox / OPNsense / TrueNAS / Caddy 等
- **技術ブログ・記事の実践検証**：問題解決のために複数の情報源を参照
- **AI活用**：ChatGPT / Claude を使った技術調査・コード生成
- **実践による学習**：障害対応を通じた深い理解
- **ドキュメント化**：Obsidian で学習内容を体系的に記録

### 特に参考にした分野

- Proxmox HA / クラスタ構成
- ZFS レプリケーション戦略
- OPNsense VXLAN / ファイアウォール設計
- Ansible による Infrastructure as Code
- Docker Compose によるコンテナ運用

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