---
layout: default
title: 中村雄介 | インフラ/バックエンドエンジニア ポートフォリオ
---

# 中村雄介

Yusuke Nakamura  
（2026年 求職中・フレックス/ハイブリッド希望）

技術スキルの維持と向上、そして実用目的のために  
**3ノード Proxmox VE クラスタを中核としたプライベートクラウド環境**を構築し、  
OS / ネットワーク / ストレージ / セキュリティ / 監視 / 自動化 を横断的に学習・運用してきました。

---

# 👤 プロフィール
 
- 学歴：電気系修士（2024年修了）  
- 元職種：車載空調制御（制御設計職）  
- 志望：**主にインフラ / バックエンドで応募中**  
- 働き方：**ハイブリッド/リモート勤務・フレックス希望**

---

# 🧠 主要スキル

### バックエンド（主軸）
- TypeScript / Node.js、Fastify（API実装）
- REST API設計（認可 / 検証 / 監査ログ）
- zod（contracts / 実行時バリデーション）
- vitest（テスト）
- monorepo（pnpm workspace）
- ADRによる意思決定文書化

### インフラ（運用経験）
- Proxmox VE 8.x（3ノード/クラスタ）、KVM / LXC
- OPNsense HA（CARP + pfSync）、VXLAN（Proxmox SDN）
- TrueNAS SCALE + ZFS（ミラー / レプリケーション / NFS）
- Caddy / Step-CA（証明書自動更新）、Discord Webhook 監視通知
- Ansible / Cloud-init（VM自動構築）

---

# 💻 開発プロジェクト：Yomicord

技術学習として、**Discord音声読み上げBot**を設計・開発中。  
インフラ運用で培った「運用を見据えた設計」をアプリケーション開発に応用しています。

**技術スタック**: TypeScript, Node.js, Fastify, Zod, discord.js, pnpm, Vitest | **開発状況**: 設計・初期実装フェーズ

### 設計アプローチ

**アーキテクチャ**: Single Writer パターン + Contracts First（Zod）、責務分離、ADR文書化  
**AI活用**: GitHub Copilot/Codex を品質ガードレール下で運用（二層ルール・境界検証・差分管理）

**2026年の差別化**: 「AIを使える」ではなく「品質を保ちながら制御して使える」

➡️ GitHub リポジトリ: <https://github.com/Hakuya5247/yomicord>  
➡️ プロジェクト詳細: [Yomicord開発プロジェクト](/details/yomicord.html)

---

# 🏗️ Home Lab 概要（3ノード Proxmox クラスタ）

学習基盤として、**小規模ながら商用レベルに近いプライベートクラウド環境**を構築しました。

**運用規模**
- VM/CT数：19台
- VXLANセグメント：6つ
- NASストレージ：ZFSミラー 4TB × 2
- NASレプリケーション：日次/週次/月次管理

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

- **運用期間**: 4ヶ月（2025年9月〜）
- **運用規模**: VM/CT 19台、VXLANセグメント 6つ、ZFSミラー 4TB × 2
- **安定性**: 4ヶ月の集中運用の中で最長連続稼働60日以上、HA構成により障害時も無停止運用を実現

---

# 🌐 ネットワーク・ストレージ構成

- **Proxmox SDN + VXLAN** による6セグメント論理分離
- **OPNsense HA**（CARP + pfSync）による完全冗長ルーター
- ゾーンベースの**ゼロトラスト設計**
- **TrueNAS SCALE** によるZFSミラー構成（4TB × 2）
- **ZFS レプリケーション**によるNASデータの日次/週次/月次バックアップ

→ [インフラ構成の詳細を見る](/details/infrastructure.html)

---

# 🧱 サービス一覧

**運用中の主要サービス（8種）**：OPNsense（HA構成）、TrueNAS、Step-CA、Caddy、Nextcloud、Immich、Forgejo、Pulse

→ [サービス一覧の詳細を見る](/details/services.html)

---

# 💾 バックアップ・DR戦略

**3層バックアップ構成**：ZFSスナップショット（15分〜2時間毎）+ Proxmoxバックアップ（日次/週次/月次）+ ZFSレプリケーション  
**災害復旧指標**：RPO 24時間以内 / RTO 2時間以内

→ [バックアップ・監視の詳細を見る](/details/operations.html)

---

# 🔍 監視・運用体制

**Pulse**（Proxmox専用監視）によるクラスタ全体の可視化、Discord Webhookによる即時アラート通知、定期メンテナンス（SMART監視週1回）

---

# 🔒 セキュリティ対策

**ゼロトラストアーキテクチャ**（6セグメント完全分離）、3層ファイアウォール、TLS 1.3 / SSH公開鍵認証 / 2FA、Step-CA / Let's Encrypt による証明書自動管理

→ [セキュリティ詳細を見る](/details/operations.html)

---

# 🔥 障害対応例

### Immich Android アプリが内部 CA を信頼しない

**症状**：Step-CA で発行した証明書を Android アプリが拒否  
**原因**：Androidアプリ側の仕様で追加CAを信頼しない  
**対策**：Caddy 側の証明書を **Let's Encrypt** へ切り替え（DNS-01チャレンジ / Cloudflare API連携）  
**結果**：アプリ互換性とセキュリティを両立、外部からのアクセスも可能に

→ [その他の障害対応例を見る](/details/operations.html)

---

# 🎯 技術選定の理由

自宅環境の制約（VLAN対応スイッチなし、既存ネットワーク活用）と学習目的を考慮し、オープンソース・HA対応・WebUI充実 を基準に選定。

→ [技術選定の詳細を見る](/details/infrastructure.html)

---

# ⚙️ 自動化（Cloud-init + Ansible）

**Cloud-init + Ansible** によるVM自動構築を実現。構築時間を **手動1時間 → 自動10分** に短縮、再現性を確保。

➡️ GitHub リポジトリ: <https://github.com/yusuke-524/ansible-immich>

---

# 🚀 今後の学習・キャリア方向

### ✔ インフラエンジニア
- Prometheus + Grafana（Pulse に加えて検討中）  
- Terraform による IaC

### ✔ クラウドインフラ
- AWS での PoC（EC2 / S3 / RDS 等）  
- マルチクラウド構成の検証

### ✔ バックエンドエンジニア
- yomicordプロジェクト

---

# 📬 連絡先

- **GitHub**: <https://github.com/yusuke-524>  
- **ポートフォリオ**: <https://yusuke-524.github.io>
