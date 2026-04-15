---
name: sisakulint
description: >
  sisakulint による GitHub Actions ワークフローのセキュリティ硬化。
  Trigger: sisakulint, ワークフロー hardening, CI/CDセキュリティ, actions pinning, workflow lint
---

# sisakulint

GitHub Actions ワークフローの静的セキュリティリンター。52ルールで脆弱性を検出し、38+ルールで自動修正する。

## 手順

1. `sisakulint` を実行しスキャン開始。command not found の場合は `go install github.com/sisaku-security/sisakulint/cmd/sisakulint@latest` でインストールしてリトライ
2. 検出結果を確認し `sisakulint -fix on` で自動修正を適用
3. GitHub APIレート制限で commit-sha 解決に失敗した場合、`sisakulint -fix dry-run` 出力のSHAを参考に手動で残りを修正
4. artipacked ルール対策として `actions/checkout` に `persist-credentials: false` を追加（自動修正されない場合がある）
5. `sisakulint` を再実行し全ルールクリアを確認

リモートリポジトリの場合は `sisakulint -remote owner/repo` でスキャンし、ローカルにcloneして修正する。

## 手動対応が必要なルール

| Rule | 検出内容 | 手動対応 |
|------|---------|---------|
| commit-sha | タグ参照をフルSHAにピン | dry-run出力のSHAをコピー（API制限時） |
| artipacked | checkout の認証情報永続化 | `persist-credentials: false` を追加 |
| dependabot-github-actions | dependabot設定の欠如 | `.github/dependabot.yaml` を作成 |
| code-injection-critical | 特権トリガーでのコードインジェクション | 式展開を中間変数に置換 |
| untrusted-checkout | 未信頼PRコードのcheckout | `pull_request_target` のref見直し |

## ルール全量リファレンス

52ルールの完全なリストと詳細は [references/rules.md](references/rules.md) を参照。
