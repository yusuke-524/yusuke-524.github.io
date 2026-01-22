---
layout: default
title: インフラ構成詳細
---

← [トップに戻る](/)

# インフラ構成詳細

---

## 🌐 ネットワーク構成（VXLAN + OPNsense HA）

### セグメント設計

Proxmox SDN + VXLAN による論理分離（MTU 1450）

| セグメント | CIDR         | 用途                     | 備考              |
| ---------- | ------------ | ------------------------ | ----------------- |
| manage     | 10.0.10.0/24 | サーバー管理・証明書配布 | Step-CA配置       |
| remote     | 10.0.11.0/24 | リモートアクセス         | Cloudflare tunnel |
| nas        | 10.0.20.0/24 | ストレージ               | TrueNAS / NFS     |
| service    | 10.0.30.0/24 | 個人サービス             | Caddy Proxy配下   |
| family     | 10.0.40.0/24 | 家族向けサービス         | AdGuard Home      |
| public     | 10.0.50.0/24 | 外部公開                 | Minecraft等       |

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

## 💽 ストレージ構成（TrueNAS）

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

## 🎯 技術選定の理由

| 技術              | 採用理由                                                                                    | 検討した代替案                     |
| ----------------- | ------------------------------------------------------------------------------------------- | ---------------------------------- |
| **Proxmox VE**    | ・KVM/LXC 両対応<br>・WebUI が充実<br>・クラスタ・HA機能が標準<br>・オープンソース          |                                    |
| **OPNsense**      | ・HA機能（CARP/pfSync）<br>・VXLAN 対応<br>・FreeBSD ベースの安定性<br>・WebUI の使いやすさ | pfSense, VyOS, OpenWrt             |
| **TrueNAS SCALE** | ・ZFS の信頼性<br>・レプリケーション機能<br>・NFS/SMB 統合管理<br>・WebUI                   | FreeNAS Core                       |
| **VXLAN**         | ・VLAN対応スイッチ不要<br>・既存ネットワーク上に構築可<br>・柔軟なセグメント設計            | 物理VLAN, Wireguard メッシュ       |
| **Caddy**         | ・自動TLS更新<br>・Caddyfile のシンプルさ<br>・DNS-01チャレンジ対応                         | Nginx, Traefik, HAProxy            |
| **Ansible**       | ・エージェントレス<br>・冪等性の保証<br>・YAML の可読性<br>・学習コスト低                   | Terraform                          |
| **Step-CA**       | ・プライベートCA<br>・ACME対応<br>・軽量（コンテナで動作）                                  | 自己署名証明書, Let's Encrypt のみ |

---

← [トップに戻る](/)
