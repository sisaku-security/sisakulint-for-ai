# sisakulint Rules Reference (52 rules)

## Code Injection & Expression Safety (9)

| ID | Severity | Detection |
|----|----------|-----------|
| code-injection-critical | Critical | 特権トリガーでのコードインジェクション |
| code-injection-medium | Medium | 通常トリガーでのコードインジェクション |
| envvar-injection-critical | Critical | 高リスクトリガーでの環境変数インジェクション |
| envvar-injection-medium | Medium | 通常トリガーでの環境変数インジェクション |
| envpath-injection-critical | Critical | 特権コンテキストでのPATH操作 |
| envpath-injection-medium | Medium | 通常コンテキストでのPATH操作 |
| argument-injection | High | コマンドライン引数インジェクション |
| output-clobbering | High | $GITHUB_OUTPUT経由のoutput上書き |
| unsound-contains | Medium | 条件式での安全でないcontains()使用 |

## Supply Chain & Dependency Security (7)

| ID | Severity | Detection |
|----|----------|-----------|
| commit-sha | High | アクション参照のフルSHA使用を検証 |
| known-vulnerable-actions | Critical | 既知の脆弱性を持つアクションの検出 |
| archived-uses | Medium | アーカイブ済み/非推奨アクションの検出 |
| impostor-commit | Critical | なりすましコミット攻撃パターン |
| ref-confusion | High | ブランチ参照操作の脆弱性 |
| unpinned-images | High | バージョン固定されていないコンテナイメージ |
| action-list | Medium | アクション許可/拒否リスト強制 |

## Credential & Secret Protection (7)

| ID | Severity | Detection |
|----|----------|-----------|
| credentials | Critical | ハードコードされた認証情報パターン（Rego） |
| secret-exposure | High | 過剰なシークレット公開 |
| unmasked-secret-exposure | High | ワークフローログでのマスクされていないシークレット |
| secret-exfiltration | High | ネットワーク経由のシークレット窃取 |
| secrets-in-artifacts | High | アップロードアーティファクト内の機密データ |
| secrets-inherit | Medium | 過剰なシークレット継承 |
| artipacked | Medium | 認証情報永続化の脆弱性パターン |

## Pipeline Poisoning & Artifact Integrity (8)

| ID | Severity | Detection |
|----|----------|-----------|
| untrusted-checkout | Critical | 未信頼PRコードのcheckout |
| untrusted-checkout-toctou-critical | Critical | checkout時のTOCTOU競合（Critical） |
| untrusted-checkout-toctou-high | High | checkout時のTOCTOU競合（High） |
| artifact-poisoning-critical | Critical | 重要フローでの悪意あるアーティファクト注入 |
| artifact-poisoning-medium | Medium | 通常フローでのアーティファクト改ざん |
| cache-poisoning | High | キャッシュ汚染の脆弱性 |
| cache-poisoning-poisonable-step | High | 脆弱なcheckout後の安全でないステップ |
| reusable-workflow-taint | High | 再利用ワークフローでの未信頼入力 |

## Triggers & Access Control (7)

| ID | Severity | Detection |
|----|----------|-----------|
| dangerous-triggers-critical | Critical | 緩和策なしの特権トリガー |
| dangerous-triggers-medium | Medium | 部分的緩和策のある特権トリガー |
| permissions | High | パーミッションスコープと値の検証 |
| bot-conditions | Medium | ワークフロー内のbotアクター条件の検証 |
| improper-access-control | High | ラベルベースの承認バイパスパターン |
| self-hosted-runners | Medium | セルフホストランナーのセキュリティ検証 |
| request-forgery | High | SSRF脆弱性 |

## AI Agent Security (3)

| ID | Severity | Detection |
|----|----------|-----------|
| ai-action-unrestricted-trigger | Critical | 任意ユーザーが実行可能なAIアクション |
| ai-action-excessive-tools | High | 未信頼トリガーでのAIエージェントへの危険なツール付与 |
| ai-action-prompt-injection | High | AIエージェントプロンプトへの未信頼入力の補間 |

## Workflow Quality & Best Practices (11)

| ID | Severity | Detection |
|----|----------|-----------|
| id | Medium | ジョブ・環境変数のID衝突 |
| timeout-minutes | Medium | timeout-minutes未設定 |
| workflow-call | Medium | 再利用ワークフロー呼び出しの検証 |
| conditional | Medium | 条件式の検証 |
| deprecated-commands | Low | 非推奨ワークフローコマンド |
| environment-variable | Low | 環境変数名の検証 |
| job-needs | Medium | ジョブ依存関係の検証 |
| cache-bloat | Low | キャッシュサイズの効率性 |
| obfuscation | Medium | ワークフロー内の難読化コード |
| expression | Medium | GitHub Actions式構文の検証 |
| dependabot-github-actions | Medium | Actionsエコシステム用Dependabot設定 |
