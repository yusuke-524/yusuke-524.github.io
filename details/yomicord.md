---
layout: default
title: Yomicord - バックエンド開発プロジェクト
---

← [トップに戻る](/)

# Yomicord - Discord読み上げBot

**開発状況: 設計・初期実装フェーズ**

---

## 📋 プロジェクト概要

Discord のチャットを音声で読み上げるBotのバックエンドAPI。  
将来的にWebUIからも設定管理できる運用を想定し、  
**整合性・監査・拡張性を重視した設計**を行っています。

**背景と目的**
- 療養期間中の技術学習の一環として開発
- インフラ運用で培った「運用を見据えた設計」をアプリケーション開発に応用
- 複数クライアント（Bot / WebUI）を想定したバックエンドAPI設計の実践

**技術的な挑戦**
- 設計フェーズを重視し、実装前にアーキテクチャを詳細に検討
- ADR（Architecture Decision Records）による意思決定の文書化
- AI活用における品質管理とガードレールの構築

---

## 🎯 設計方針（重要な判断）

### 1. Single Writer パターン
- **DB への読み書きは API のみが行う**
- Bot/WebUI は API 経由でのみデータを操作
- **効果**: データ整合性の保証、認可ロジックの一元管理、監査ログの確実な記録

### 2. Contracts as Source of Truth
- **API 入出力は packages/contracts に定義された schema（zod）を正とする**
- すべてのクライアントが同一の型定義を共有
- **効果**: 型安全性、クライアント間の一貫性、実行時バリデーション

### 3. 責務分離
- **API**: 認可・検証・整合性・監査ログ
- **Bot**: Discord UI/UX と読み上げ
- **(将来) WebUI**: 管理画面
- **効果**: 保守性・テスタビリティの向上、各コンポーネントの独立性

---

## 🏗️ アーキテクチャ

```
yomicord/ (monorepo - pnpm workspace)
├─ apps/
│  ├─ api/           # Backend API（唯一のDB窓口）
│  │  ├─ src/
│  │  │  ├─ app.ts                 # Fastify 初期化・ルート登録
│  │  │  └─ app/
│  │  │     ├─ routes/             # 機能別ルート定義
│  │  │     │  ├─ guild-settings.ts
│  │  │     │  ├─ guild-member-settings.ts
│  │  │     │  ├─ dictionary.ts
│  │  │     │  ├─ audit-logs.ts
│  │  │     │  └─ health.ts
│  │  │     ├─ http/               # HTTP境界の共通処理
│  │  │     ├─ audit-log.ts        # 監査ログ横断処理
│  │  │     └─ internal/           # 依存注入型定義
│  │  └─ package.json
│  └─ bot/           # Discord Bot（API経由で操作）
│     ├─ src/
│     │  ├─ index.ts
│     │  └─ readyHandler.ts
│     └─ package.json
├─ packages/
│  ├─ contracts/     # API入出力 schema（zod）
│  │  └─ src/
│  │     ├─ guild/                 # Guild関連の型・schema
│  │     ├─ dictionary/            # 辞書エントリ
│  │     ├─ audit-log/             # 監査ログ
│  │     ├─ common/                # 共通型（Actor等）
│  │     └─ index.ts
│  ├─ storage/       # Store interface（境界）
│  │  └─ src/
│  │     └─ interfaces/            # 永続化方式に依存しないinterface
│  └─ storage-json/  # JSON実装（初期フェーズ）
│     └─ src/
│        └─ json-store.ts          # atomic write実装
└─ docs/
   ├─ architecture/                # アーキテクチャドキュメント
   │  ├─ overview.md
   │  ├─ data-flow.md
   │  └─ roadmap.md
   └─ adr/                         # 意思決定記録（5件）
      ├─ 0001-guild-settings-api.md
      ├─ 0002-guild-member-settings-api.md
      ├─ 0003-dictionary-entry-api.md
      ├─ 0004-settings-audit-log-api.md
      └─ 0005-settings-audit-log-write.md
```

---

## 🤖 AI活用とガードレール（特徴的な取り組み）

### 基本方針: 制御されたAI活用

開発効率化のため **GitHub Copilot** と **OpenAI Codex** を全面活用しています。  
ただし、**品質・安全性・整合性を担保するガードレール体制**を構築し、  
闇雲なAI依存ではなく、エンジニアが制御する形での活用を徹底しています。

### 二層のガードレール体制

**1. 文書レベルのルール**
- **AGENTS.md**: AI運用ルール全般（正）
  - 202行の詳細な運用ルール
  - GitHub Copilot Agent モードおよび OpenAI Codex の両方に適用
  - 作業範囲・出力形式・確認質問のテンプレート化
- **.github/copilot-instructions.md**: Copilot専用の生成時ルール
  - 恒久的に守るべき境界・契約・安全性ルール
  - コメント・ログ文・ドキュメントは日本語を基本とする方針

**2. スキルレベルの強制ルール**（.codex/skills/配下に10種類）
- **repo-guardrails**: DB境界違反・依頼範囲外変更の検知
  - データベース直アクセス禁止の強制
  - 依頼されていないファイル変更の禁止
  - 差分200行超過で自動停止
- **contract-first-api-change**: API変更の標準手順強制
  - contracts → api → bot → docs の順序を強制
  - API変更時の必須ステップをチェック
- **safe-command-policy**: 破壊的操作の事前確認
  - `rm -rf`, `git reset --hard`, `git push --force`, `sudo` 等は事前確認必須
  - 秘密情報（トークン・接続文字列）のログ・コード出力禁止
  - `.env` や鍵ファイルの新規作成・更新は実行前確認
- **plan-then-diff**: 実装前計画の提示義務
  - 変更前に2〜6ステップの計画を提示
  - 目的・変更内容・影響範囲を明確化
- **review-specialist**: 読み取り専用レビュー
  - 編集せずルール違反を検知
  - 設計判断のフィードバック
- **adr-writer**: ADR記録の自動化
  - Title / Context / Decision / Consequences / Alternatives の構成
  - 連番管理（docs/adr/配下）
- **pnpm-check-suite**: 検証コマンド定型化
  - 部分確認・最終確認の標準化
  - 影響範囲に応じた検証の実行
- その他3種類（architecture-options, design-brief, decision-checklist）

### 品質管理の仕組み

**実装前:**
- **計画提示**: 2〜6ステップの変更計画（plan-then-diff skill）
- **確認質問**（最大3つ）:
  - 期待する入出力（payload例、必須/任意、デフォルト値）
  - 失敗時の扱い（ユーザー通知、冪等性、リトライ要否）
  - 互換性（既存クライアントへの影響、移行が必要か）

**実装中:**
- **範囲制約**: 依頼範囲外の変更禁止（ついでの改善・過剰リファクタ禁止）
- **差分管理**: 差分200行超過で自動停止・確認要求
- **自動検証**: DB境界・契約ファーストの自動チェック

**実装後:**
- **レビュー**: review-specialist による読み取り専用レビュー
- **自動検証**: pnpm-check-suite による `format:check / lint / build`
- **ADR記録**: 設計判断の文書化（adr-writer skill）

### プロンプトの永続化戦略

- **devcontainer のバインドマウント**: `~/.codex` を永続化
  - プロンプト情報がコンテナ再作成後も保持される
  - `.devcontainer/devcontainer.json` で設定
- **Codex CLI の自動インストール**: `.devcontainer/postCreate.sh`
  - `npm install -g @openai/codex@latest`
- **スキル定義による知識のモジュール化**
  - YAML frontmatter付きMarkdown
  - `.codex/skills/` 配下に構造化

### AI役割分担

**Codex（ChatGPT）の役割:**
- 設計整理・計画立案
- 複数ファイル横断検討
- レビュー・フィードバック
- ADR作成支援

**GitHub Copilot の役割:**
- 実装補完・型補完
- 局所的な修正
- テストコード生成

**スペシャリストモードの切り替え:**
- **実装スペシャリスト**: 仕様に沿った最小差分実装
- **テストスペシャリスト**: 変更の検証と結果報告
- **レビュースペシャリスト**: 編集せずルール違反を検知

### セキュリティ対策

- **秘密情報の取り扱い**:
  - トークン・接続文字列・Cookie・個人情報をログ・コードに出さない
  - 秘密情報をコマンドに直書きしない（環境変数を使用）
  - `.env` や鍵ファイルの新規作成・更新は実行前確認必須
- **破壊的操作の制御**:
  - `rm -rf`, `git reset --hard`, `git push --force`, `sudo` 等は事前確認必須
  - 読み取り専用コマンド（`git status`, `git diff`, `ls`, `cat`）は自動実行可
- **監査可能性の維持**:
  - AI支援による変更も監査ログで追跡
  - ADRによる設計判断の記録

### この取り組みの意義

**2026年時点でのAI活用において重要なのは**、「使える」ことではなく、  
**「品質を保ちながら制御して使える」**ことです。

- **ガードレールによる自動的な品質管理**: 人的ミスの防止
- **プロンプトエンジニアリングの体系化**: 再現性の確保
- **チーム開発を見据えた設計**: スケーラビリティ
- **AI支援下でも追跡可能性・監査可能性を維持**: 信頼性

この体制により、AI支援下でも**安全性・一貫性・追跡可能性**を維持しています。

---

## 📝 ADR（Architecture Decision Records）

重要な設計判断をADRとして文書化し、「なぜこう設計したか」を明確にしています。

| ADR | タイトル | 概要 |
| --- | --- | --- |
| **0001** | GuildSettings API の設計 | GET/PUT全置換方式、Actor情報をカスタムヘッダーで受信 |
| **0002** | GuildMemberSettings API | GET/PUT/DELETE、本人のみ操作可、空なら自動削除 |
| **0003** | DictionaryEntry API | CRUD + cursor方式pagination、重複surfaceKeyはエラー |
| **0004** | SettingsAuditLog API | 読み取り専用、API内部で自動記録（改ざん防止） |
| **0005** | 監査ログの保存戦略 | 同期追記、変更ごとに1ログ、整合性を優先 |

**各ADRの詳細**: Context / Decision / Consequences / Alternatives の構成で記録  
➡️ [GitHubリポジトリ](https://github.com/Hakuya5247/yomicord/tree/main/docs/adr)で全文を確認できます

---

## 🔧 実装済み機能

### API基盤
- ✅ Fastify ベースのバックエンド構築
- ✅ ルート定義（5種類）
  - GuildSettings（GET/PUT）
  - GuildMemberSettings（GET/PUT/DELETE）
  - Dictionary（GET/POST/PUT/DELETE + pagination）
  - AuditLog（GET）
  - Health（GET）
- ✅ エラーハンドリング標準化
  - `{ ok: true/false, ... }` 形式
  - エラーコード体系（VALIDATION_FAILED / UNAUTHORIZED / FORBIDDEN / NOT_FOUND / CONFLICT / INTERNAL）
- ✅ Actor情報による認可・監査基盤
- ✅ テスト環境構築（vitest）

### packages/contracts
- ✅ Zod schema定義（Guild/Dictionary/AuditLog/Common）
- ✅ TypeScript型定義
- ✅ デフォルト値生成・正規化・canonicalizeヘルパー

### packages/storage
- ✅ Store interface定義（永続化方式に依存しない境界）
- ✅ JSON実装（storage-json）
  - atomic write（temp file → rename）
  - Zod validate
  - 監査ログはJSON Lines形式

### 開発環境
- ✅ monorepo構成（pnpm workspace）
- ✅ CI/CD準備（GitHub Actions）
- ✅ Dev Container対応
- ✅ Linter/Formatter設定（ESLint + Prettier）
- ✅ AI活用ガードレール（AGENTS.md + 10種類のスキル定義）

---

## 🚧 今後の実装予定

- [ ] API エンドポイントの完全実装（ビジネスロジック）
- [ ] Bot のスラッシュコマンド実装
- [ ] 音声読み上げエンジン統合（VOICEVOX）
- [ ] WebUI（管理画面）の追加
- [ ] DB移行（JSON → PostgreSQL等）
- [ ] 認証強化（JWT認証への移行）
- [ ] キャッシュ実装（Redis等）
- [ ] 変更通知機能（Bot へのキャッシュ更新通知）

---

## 💡 学習・技術的チャレンジ

### 設計面
- **複数クライアントを想定したAPI設計**: Bot/WebUIが同一APIを共有する前提での設計
- **整合性・監査を考慮したアーキテクチャ**: Single Writer、監査ログ自動記録
- **ADRによる意思決定の可視化**: 設計判断を文書として残す重要性

### 技術面
- **zod による実行時型検証**: TypeScript型とzod schemaの両立
- **monorepo による依存関係管理**: pnpm workspaceの活用
- **cursor方式のpagination設計**: `{priority}:{surfaceLength}:{id}` のbase64化

### 開発プロセス
- **実装前の詳細設計フェーズの重要性**: アーキテクチャを先に固めることで後の実装がスムーズに
- **ドキュメントファーストアプローチ**: 設計ドキュメント・ADRを先に書く
- **拡張性を考慮した境界設計**: Storage境界でDB移行を見据える

### AI活用における学び
- **ガードレールの重要性**: 自動検証により品質を担保
- **プロンプトエンジニアリングの体系化**: スキル定義による再現性の確保
- **AI支援下でも監査可能性を維持**: 変更履歴・設計判断を明確に記録

---

## 🔗 リンク

- **GitHub リポジトリ**: <https://github.com/Hakuya5247/yomicord>
- **技術スタック**: TypeScript, Node.js, Fastify, Zod, discord.js, pnpm, Vitest
- **開発環境**: Dev Container, ESLint, Prettier, GitHub Actions

---

← [トップに戻る](/)
