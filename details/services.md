---
layout: default
title: サービス・ドキュメント詳細
---

← [トップに戻る](/)

# サービス・ドキュメント詳細

---

## 🧱 サービス一覧（主要）

| サービス      | 概要             | 技術ポイント                                 | 配置           |
| ------------- | ---------------- | -------------------------------------------- | -------------- |
| **OPNsense**  | ルーター         | CARP/pfSync HA / VXLAN統合 / Wireguard       | pvecore + pve1 |
| **TrueNAS**   | NAS              | ZFS / レプリケーション / NFS                 | pvecore        |
| **Step-CA**   | プライベートCA   | ACME / HTTP-01                               | CT(manage)     |
| **Caddy**     | Reverse Proxy    | DNS-01 / Let's Encrypt / Cloudflare API      | CT(service)    |
| **Nextcloud** | ファイル同期     | NFS + Caddy + TLS                            | VM(nas)        |
| **Immich**    | 写真バックアップ | Docker Compose + NFS + 自動デプロイ(Ansible) | VM(nas)        |
| **Forgejo**   | Git ホスティング | Reverse Proxy + TLS                          | CT(service)    |
| **Pulse**     | Proxmox 監視     | クラスタ全体可視化 / Discord通知             | CT(manage)     |

---

## 📚 ドキュメント管理体制

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
└─自動化/
  ├ cloud-init.md
  └ Ansible-immich.md
```

### 設定ファイル管理（Forgejo / Git）

- Ansible Playbook
- Docker Compose ファイル
- システム設定ファイルのバージョン管理
- 変更履歴の追跡

### Obsidian 同期

- Forgejo + Gitプラグインによる複数デバイス間の同期
- 自宅サーバー内で完結（外部サービス不使用）

---

## 📖 学習方法・リソース

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

← [トップに戻る](/)
