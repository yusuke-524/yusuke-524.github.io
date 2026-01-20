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

← [トップに戻る](/)
