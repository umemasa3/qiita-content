# 記事のネタ帳

Qiitaには投稿されない（`public/` の外にあるため）。
書き始めたら「状態」を更新し、公開したら記事のファイル名を書く。

## シリーズ：AI駆動開発とPlatform Engineering

次の順で書く。後の記事は前の記事にリンクし、同じ説明を繰り返さない。

1. ハーネスの設計と運用
2. Platform Engineeringチームが整備する範囲
3. RHDHでAI駆動開発のGolden Pathを提供する

---

### 1. コーディングエージェントを1つに統一できない組織のための、ハーネスの設計と運用

- 状態: 執筆中
- ファイル: `public/agent-agnostic-harness.md`
- 前提の記事: `ai-driven-dev-needs-harness.md`（続編として書く）
- 読者: 大規模プロジェクトでチームごとに使うエージェントが違う組織のエンジニア

要点:

- 品質をそろえる仕組みはエージェントの外側に置く。最終的な品質は、AIが書いたか人が書いたかに関係なく通るゲートで担保する
- ハーネスは3層。品質ゲート（CI、ブランチ保護）、指示と知識（AGENTS.md、Skills、MCP）、実行の統制（各ツールの管理機能）
- 指示と知識の層を実装する道具の紹介。AI-DLCを詳しく、ほかは役割（進め方をそろえる／ルールを配る）で分けた表で
- AI-DLCのsensorsは記録するだけで作業を止めない。品質ゲートの代わりにはならない
- 管理の仕方（置き場所と責任者、全社→チーム→リポジトリの上書き、バージョンと配布）
- 効果が出ているかの確かめ方（代表タスクで比べる、エージェント別のCI失敗率）

図: 3層の構成図

### 2. Platform EngineeringチームはAI駆動開発をどこまで整備すべきか

- 状態: 構想
- 前提の記事: 1、`ai-coding-agent-governance-comparison.md`、`kiro-dotkiro-standardization.md`

構成案:

1. 整備しすぎた場合と、しなさすぎた場合に起きること
2. 線引きの基準。ばらつくと全社に影響が出るものは整備し、チームの生産性だけに関わるものは任せる
3. 層ごとの担当
   - Platform Engineeringチーム: ID、ガバナンス、品質ゲート、共通のMCPサーバー、利用状況の計測
   - 各チーム: チーム固有の規約、Skills、使うエージェントの選択（許可された中から）
4. 段階的な広げ方
5. まとめ

### 3. AI駆動開発のGolden Pathを、Platform Engineeringチームはどう用意するか

- 状態: 構想
- 前提の記事: 2、`rhdh-software-template-granularity.md`、`rhdh-approval-workflow-patterns.md`

構成案:

- RHDHのSoftware Templateで、AGENTS.md、Skills、CIの設定が入った状態のリポジトリを作れるようにする
- 2で「Platform Engineeringチームが整備する」とした部分を、どうテンプレートに入れるか
- 既存のリポジトリへの後から入れ方（同期PRを出すbotなど）

注意: Backstage/RHDHを使っていない読者には遠い内容になるので、冒頭で前提を書く

---

## その他のネタ

### AI-DLCを1つのリポジトリで複数のエージェントから動かしてみる

- 状態: 保留（実際に動かす検証が必要）
- 同じ作業をClaude Code・Codex・Kiroから依頼し、成果物の違いとCIを1回で通ったかを比べる

### Lambda MicroVMsでGitLab Runnerを動かす場合とEC2の比較

- 状態: 保留
- 公式のexecutorがないこと、ARM64のみ、`services:`が使えない、イメージの脆弱性スキャンの問題
