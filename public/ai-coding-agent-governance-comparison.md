---
title: "AIコーディングエージェント5製品を組織で使うとき、ガバナンスが一番効かせやすいのはどれか"
tags:
  - "AIエージェント"
  - "ガバナンス"
  - "AI駆動開発"
  - "エンタープライズ"
  - "セキュリティ"
private: false
updated_at: ""
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

AIコーディングエージェントを組織に導入する検討をしていると、必ず出てくる質問があります。

「どのツールが一番ガバナンスを取りやすいの？」

主要な製品はどれも「組織向けの管理機能がある」とうたっています。
しかし、管理の入口、ポリシーの届け方、効く範囲、迂回されやすさはかなり違います。
この記事では、次の5製品を各社の公式ドキュメント（2026年11月時点）をもとに比較します。
並びはアルファベット順で、優劣とは関係ありません。

- **Antigravity**（Google。Gemini Enterprise経由の利用）
- **Claude Code**（Anthropic）
- **Codex**（OpenAI。ChatGPT Enterprise経由の利用）
- **GitHub Copilot**（GitHub）
- **Kiro**（AWS）

比較の軸は、**管理者がガバナンスを効かせるまでの手間と、効かせた後の確実さ**です。

:::note info
この記事の「容易さ」は、機能の多さではなく「管理者が少ない手順で、組織の大多数に、迂回されにくい形でポリシーを適用できるか」を指します。
機能が多いほど設計すべき項目も増えるため、多機能であることと容易であることは必ずしも一致しません。
:::

:::note info
対象は、開発者の端末で動く使い方（IDE、CLI、デスクトップアプリ）です。
Devinのように、ベンダーのクラウド上で動くことを前提にしたエージェントは、比べるべき観点が違うため対象外にしています。
:::

参考:

- Antigravity: [Configure AI developer tools settings](https://docs.cloud.google.com/gemini/enterprise/docs/ai-developer-tools-settings)
  / [Enterprise gateway](https://docs.cloud.google.com/gemini/enterprise/docs/ai-developer-tools/deploy-enterprise-gateway)
- Claude Code: [Admin setup](https://code.claude.com/docs/en/admin-setup)
  / [Server-managed settings](https://code.claude.com/docs/en/server-managed-settings)
- Codex: [Admin rollout guide](https://learn.chatgpt.com/docs/enterprise/admin-setup)
  / [Managed configuration](https://learn.chatgpt.com/docs/enterprise/managed-configuration)
- GitHub Copilot: [Copilot policies](https://docs.github.com/en/copilot/concepts/policies)
  / [Enterprise-managed settings](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/use-managed-settings/get-started)
- Kiro: [Governance](https://kiro.dev/docs/enterprise/governance/)
  / [Enterprise Management](https://kiro.dev/docs/enterprise/enterprise-management/getting-started/)

## 結論

どれが一番容易かは、**組織がすでに何を持っているか**でほぼ決まります。

| 組織の前提 | 効かせやすいもの | 理由 |
|-----------|----------------|------|
| Google Cloud / Gemini Enterpriseを使っている | Antigravity | Gemini Enterpriseの設定画面で、セキュリティ・ログ・モデルをまとめて管理できる |
| MDMで端末を管理しており、実行内容まで細かく縛りたい | Claude Code | 制御項目が最も多く、MDM配布と組み合わせたときの強制力が高い |
| ChatGPT Enterpriseを全社導入済みで、SCIM/RBACが整っている | Codex | 既存のワークスペース、グループ、ロールにそのままポリシーを割り当てられる |
| GitHub Enterpriseを使っている | GitHub Copilot | ポリシーをリポジトリで管理でき、変更をPRでレビューして履歴を残せる |
| AWSが中心で、IAM Identity Centerを運用している | Kiro | 既存のAWSアカウントとIdCの上で、コンソールのトグル中心に設定が終わる |

そのうえで、5製品に共通する事実が2つあります。

- **個人アカウントでの利用を確実に封じるには、端末かネットワークでの対策が要る**。管理コンソールの設定だけでは封じられない
- **シェルやファイル操作の制御は、どれもクライアント側の強制が中心**。端末の管理者権限があれば迂回され得る

つまり、どの製品を選んでも「コンソールで設定すれば終わり」にはなりません。
この前提を押さえたうえで、各製品の違いを見ていきます。

## 比較の観点

ガバナンスを「効かせる」までの流れを、次の6つに分けて比較します。

1. **管理の入口** — どこで、誰が設定するか
2. **ID・サインインの統制** — 組織のIDでだけ使わせられるか
3. **ポリシーの配布方法** — 端末にどう届くか
4. **制御できる範囲** — モデル、MCP、シェル、ファイル、ネットワークなど
5. **グループ単位の差分** — 部門ごとに例外を持たせられるか
6. **可視化と監査** — 使われ方を後から追えるか

## Antigravity：Google Cloudの設定画面でまとめて管理する

### Gemini Enterpriseの設定画面

Antigravityを組織で管理するには、**Gemini Enterprise**（Standard / Plus / Pay-as-you-go）経由で使います。
設定はGoogle Cloudコンソールの Gemini Enterprise > Settings > **AI developer tools** で行い、必要なロールはGemini Enterprise Adminです。
開発者はGoogle CloudのSSO、Workforce Identity Federation（WIF）、ADCでサインインし、プロジェクトに紐づくライセンスを選びます。
利用できる人の制御はIAMのロールで行い、AI developer toolsだけを使わせないカスタムロールも作れます。

設定できる主な項目と、そのデフォルトは次のとおりです。

| 項目 | デフォルト | 管理者ができること |
|------|-----------|-----------------|
| 作業フォルダ外のファイルアクセス | 拒否 | 拒否 / 毎回確認 / 許可 |
| ターミナルの自動実行 | 常に実行 | 要レビュー / サンドボックス内で実行 / 常に実行 |
| サンドボックスモード | 無効 | 有効化 |
| ブラウザのアクセス制御 | 無効 | 許可URLリスト、JavaScript実行の可否 |
| MCPサーバー | 無効 | 許可するサーバーをJSONで定義 |
| モデル | Geminiモデルはすべて利用可 | 許可リストで制限。他社モデル（Claude）は管理者の同意が必要 |
| プロンプトと応答のログ | 無効 | 有効化（自社プロジェクトのCloud Loggingに記録） |

ログは自社のGoogle Cloudプロジェクトに保存されるため、IAMで閲覧者を絞り、BigQueryに流して分析できます。
エージェントの操作を後から追う仕組みとしては、5製品の中でも扱いやすい部類です。

### 初期値と設定の単位に注意

- **ターミナルの自動実行が初期状態で「常に実行」**です。MCPやブラウザは初期でオフですが、ここは管理者が明示的に絞る必要があります
- 設定は**プロジェクトとロケーション単位**です。公式ドキュメントで確認できた範囲では、同じプロジェクト内でグループごとに設定を変える仕組みは見当たりませんでした。部門ごとに変えたい場合はプロジェクトを分けることになります
- JetBrainsやAndroid StudioなどACPで接続するIDEでは、IDE側が独自にMCPツールを注入できるため、MCPの制御は**ベストエフォート**と明記されています
- 個人のGoogleアカウントでのサインインを禁止する設定は、公式ドキュメントで見つけられませんでした

最後の点の回避策として、**Enterprise Gateway**という構成があります。
MDMで端末に`admin_settings.json`を配り、Antigravityの通信を自社のゲートウェイ経由にする方式です。
このモードでは開発者がGoogleアカウントでサインインせず、使えるモデルも`admin_settings.json`で固定できます。
ただし、ゲートウェイの構築・運用が必要になるうえ、課金がライセンスに含まれるクレジットではなく従量課金になります。

### Antigravityの評価

**Google Cloudをすでに使っている組織にとっては、主要な設定が1画面に集まっていて導入しやすい**製品です。
ログの保管先や閲覧権限を、既存のGoogle Cloudの仕組みでそのまま管理できる点も強みです。

一方で、グループ単位で差分を持たせにくいこと、個人アカウントの封じ込めにゲートウェイ構成が要ることは、大きな組織では設計上の制約になります。

## Claude Code：制御は最も深い。そのぶん設計量も多い

### 管理の入口とポリシーの届け方

Claude Codeでは、組織のポリシーを**managed settings**として配布します。
managed settingsはユーザー設定やプロジェクト設定より優先されます。
届け方は4種類あり、優先度の高い順に次のとおりです。

| 配布方法 | 届け方 | 特徴 |
|---------|-------|------|
| Server-managed | claude.aiの管理コンソール | 起動時に取得し、1時間ごとに更新。端末側の準備が不要 |
| plist / HKLMレジストリ | MDM、グループポリシー | 書き換えに管理者権限が必要で、改ざんに強い |
| ファイル | `managed-settings.json`をシステムディレクトリに配置 | MDMのない端末やLinux向け |
| HKCUレジストリ | Windowsのユーザーレジストリ | ユーザーが書き換えられるため、強制手段にはならない |

Server-managed settingsを使うにはClaude for TeamsかEnterpriseのプランが必要で、編集できるのはOwner / Primary Ownerだけです。

### 制御できる範囲

制御項目の多さは5製品の中で最も多いです。主なものを挙げます。

- ツール・コマンドの許可/拒否ルール。ルールの出どころをmanaged settingsだけに限定することもできる
- `--dangerously-skip-permissions`（確認なしモード）の無効化
- OSレベルのサンドボックスと、通信先ドメインの許可リスト
- MCPサーバーの許可/拒否と、管理者が配ったサーバーだけに限定する設定
- プラグインマーケットプレイスの制限、ユーザー・プロジェクト由来のskills/hooks/MCPの禁止
- hooksを管理者定義のものだけに限定
- ログイン方法とログイン先組織の強制、利用できるAPIプロバイダーの制限
- モデルの制限、effortの上限、Claude Codeの最低バージョンや許可するバージョン範囲

「Bashで`curl`を許すとWebFetchを禁止しても意味がない」といった抜け道を、サンドボックスのネットワーク制限で塞ぐ方法まで公式に説明されています。
実行内容まで細かく統制したい組織には、最も応えられる製品です。

### 迂回経路が多い

制御が深い一方で、**Server-managed settingsが届かないケース**がいくつもあります。

- Server-managed settingsはクライアント側の制御であり、セキュリティ境界ではないと公式に書かれている。管理されていない端末では、管理者権限がなくても迂回できる
- シェルで`CLAUDE_CODE_USE_BEDROCK`などのプロバイダー変数や、デフォルト以外の`ANTHROPIC_BASE_URL`を設定すると、設定の取得自体がスキップされる
- Claude ConsoleのAPIキーはclaude.aiの組織とは別組織に属するため、そのキーで動くCIジョブなどには設定が届かない
- 取得に失敗するとキャッシュで動き、キャッシュが切れると管理設定なしで起動する（`forceRemoteSettingsRefresh`で失敗時に終了させることはできる）

このため公式も、MDM管理下の端末では**端末側に配布するmanaged settingsのほうが強い保証になる**としています。
Amazon Bedrockなどのクラウドプロバイダー経由で使う場合も、自前でClaude apps gatewayを立てない限り、端末側の配布が前提です。

:::note warn
グループごとに設定を分ける機能はEnterpriseプランのみです。
しかもグループの設定同士や組織全体の設定とは**マージされず**、所属グループのうち最上位の設定だけが適用されます。
全員に効かせたい制限は、すべてのグループ設定とcatch-all設定に重複して書く必要があります。
:::

### Claude Codeの評価

**「効かせられる強さ」では5製品で最も上**ですが、**「効かせるまでの容易さ」では設計量が多い**製品です。
配布経路の選択、迂回経路の封鎖（プロバイダー変数、APIキー、バージョン）、グループ設定の重複管理を、管理者が自分で設計する必要があります。
MDMとエンドポイント管理がすでに整っている組織であれば、この設計量に見合うだけの統制が得られます。

利用状況の可視化は、OpenTelemetryでの出力（全プロバイダー共通）、分析ダッシュボード、Enterprise Analytics API、Compliance APIが用意されています。
管理設定の変更履歴も、Compliance APIか監査ログのエクスポートで追えます。

## Codex：ChatGPT Enterpriseの管理体系にそのまま乗る

### ワークスペースとAgent security

Codexの組織管理は、**ChatGPT Enterpriseのワークスペース**が起点です。
メンバー、シート、SCIMで同期したグループ、RBACのロールで「誰がCodexをローカルで使えるか」「誰がCodex cloudを使えるか」を決めます。

実行時のポリシーは、Admin Console > **Agent security**で管理します。特徴は次のとおりです。

- **Requirements（ユーザーが上書きできない制約）と Defaults（ユーザーが変更できる初期値）が明確に分かれている**
- Workspace default を土台に、ユーザー・グループ・ロール単位の **Overrides** を優先度付きで割り当てられる
- 画面のフォームで編集でき、同じ内容を`requirements.toml` / `config.toml`のTOMLとしても編集・コピーできる
- 端末にファイルを配らなくても、クラウド経由で配布できる

Requirementsで制約できるのは、承認ポリシー、サンドボックス/権限プロファイル、Web検索モード、管理者定義のhooks、MCPサーバーの許可リスト、プラグインマーケットプレイス、Browser Use / Computer Useなどの機能フラグです。
OpenTelemetryの出力先や「プロンプト本文を記録しない」といった設定も、Requirementsとして強制できます。

### 境界が多く、整理に手間がかかる

Codexの難しさは、**管理の境界が多いこと**です。公式のロールアウトガイド自体が、次の境界ごとに担当者を決めるよう求めています。

- ワークスペースのアクセス（メンバー、シート、ロール）
- ローカル実行のポリシー（デスクトップアプリ、CLI、IDE拡張）
- Codex cloud（ホスト環境、リポジトリ連携）
- Platform API（APIキー認証のときは別の組織・プロジェクトの境界になる）
- プラグインとコネクタ
- 連携先システム側の権限

たとえばモデルの利用可否も、サインイン方法とサーフェスで決まり方が変わります。
APIキーで認証したCodexは、ChatGPTワークスペースではなくPlatform APIの組織・プロジェクトに従います。
Codex cloudの実行環境も、ローカル端末のポリシーやMDM設定を引き継ぎません。

:::note warn
ログイン方法やログイン先ワークスペースの制限（`allowed_login_methods`、`allowed_chatgpt_workspaces`）は、クラウド配布のRequirementsでは**無視され**、端末の`requirements.toml`かmacOSのMDMでしか効きません。
個人のChatGPTアカウントでの利用を封じるには、端末側の配布が必要です。
:::

### Codexの評価

**ChatGPT Enterpriseをすでに全社展開している組織にとっては、追加の基盤なしでグループ単位のポリシーを配れる**ため、容易さは高いです。
RequirementsとDefaultsの区別も分かりやすく、「何を強制し、何を推奨にとどめるか」を設計しやすい構造です。

一方で、ローカル/クラウド/APIの境界ごとに効く設定が違うため、「設定したつもりで効いていない」を防ぐ検証の手間は小さくありません。
可視化は、ワークスペースのCodex分析、Analytics API、Compliance APIで行います。

## GitHub Copilot：ポリシーをリポジトリで管理できる

### 2つの管理レイヤー

GitHub Copilotの管理は、2つのレイヤーに分かれています。

- **Copilotポリシー**（Enterprise / OrganizationのAI controlsタブ）。どの機能・エージェント・モデルを使えるかを決める。Enterprise側で有効・無効を決めると、Organization側では上書きできない
- **Enterprise-managed settings**（`managed-settings.json`）。クライアントの挙動を決める。権限ルール、確認なしモードの無効化、サンドボックスの強制、プラグイン、MCPの許可/拒否リストなど

特徴的なのは後者の配り方で、**Enterprise内の`.github-private`リポジトリに`copilot/managed-settings.json`を置く**と、サポートされるクライアントに約1時間で配布されます。
ポリシーの変更を、普段のコードと同じようにPRでレビューし、コミット履歴として残せます。
GitHub上でファイルの検証も行われ、エラーはAI controlsの画面に表示されます。

配布方法は3種類あり、優先度は **MDM > リポジトリ（server-managed） > ファイル > ユーザー設定** です。
ただしサンドボックスや権限ルールなど一部のキーは、どの配布方法の設定も「最も厳しい方向」に合成されます。

| 配布方法 | 対象 | 特徴 |
|---------|------|------|
| リポジトリ（server-managed） | Enterpriseからライセンスを受けたユーザー | Copilot cloud agentにも効く。Enterprise teams単位で上書きできる |
| MDM | ローカルのクライアント | ライセンスの出どころに関係なく、その端末に効く |
| ファイル | ローカルのクライアント | コンテナやCodespacesなど、MDMが使えない環境向け |

### 初期値とネットワークでの統制

管理者の手間を減らす点が2つあります。

- **Copilot coding agent（cloud agent）と外部MCPサーバーの利用は、ライセンス付与直後はオフ**です。使わせる場合だけ、管理者が明示的に有効化します
- **ネットワークのファイアウォールで、個人向けプラン（Free / Pro / Pro+など）のエンドポイントを遮断できます**。組織のプランのエンドポイントだけを許可すれば、社内ネットワークから個人のCopilotは使えません

個人アカウントの封じ込めを、端末への配布ではなくネットワーク側で実現できるのは、5製品の中でGitHub Copilotだけでした。

### 複数Organizationとサーバー配布の注意点

- 同じEnterprise内の**複数のOrganizationからライセンスを受けたユーザーには、原則として最も緩いポリシーが適用されます**。Organizationごとに設定を変えている場合は、意図せず緩くなる組み合わせがないか確認が必要です
- Server-managed settingsはサインインしたアカウントに紐づきます。CLIで取得に失敗しキャッシュもない場合、そのセッションでは適用されません。必ず効かせたい制限はMDMかファイルで配るよう、公式に案内されています
- すべてのポリシーがすべてのクライアントに効くわけではありません。対応するサーフェスの一覧で確認が必要です

### GitHub Copilotの評価

**GitHub Enterpriseを使っている組織にとっては、ポリシーをコードと同じ流れで管理できる**点が大きな利点です。
「誰が、いつ、なぜポリシーを変えたか」がPRとコミットに残るため、ポリシーが少しずつずれていくのを防ぎやすい構造です。
エージェントの操作は監査ログ（`actor:Copilot`で絞り込み、180日分）、利用状況はusage metrics APIで追えます。

一方で、Organizationの切り方によって適用されるポリシーが変わるため、組織構造が複雑な会社ほど確認の手間が増えます。

## Kiro：AWSの上で完結する。細かい制御はプレビュー段階

### Kiro consoleとIAM Identity Center

Kiroの管理は、**Kiroプロファイルを作成したAWSアカウントのKiro console**で行います。
ユーザーはIAM Identity Center（または外部IdP）経由でサブスクライブするため、既存のAWS組織とIdCを持っている会社なら、新しいID基盤を用意する必要がありません。

Settings > Shared settingsで設定できる主な項目と、そのデフォルトは次のとおりです。

| 項目 | デフォルト | 管理者ができること |
|------|-----------|-----------------|
| モデル | 制限なし（新モデルも自動で使える） | 許可リストで制限、デフォルトモデルの指定 |
| MCP | 制限なし | 全面無効化、またはMCPレジストリによる許可リスト |
| CLI用APIキー | 生成不可 | 生成を許可 |
| Web tools | 利用可 | 無効化 |
| Cloud Sessions（IdC利用時） | 無効 | 有効化 |

これらは**組織レベルで設定し、アカウント単位で上書き**できます（詳しくは[マルチアカウントの記事](https://qiita.com/umemasa3/items/6b5acecba5ed6dc9a203)で整理しました）。
APIキー生成やCloud Sessionsのように、リスクの大きい機能がデフォルトで閉じている点は、管理者にとって手間が少ない設計です。

サインイン方法の制限（Builder IDやGoogle/GitHubを使わせない）は、コンソールではなく端末に置く`managed-settings.json`で行います。
ファイルはOSの保護パス（管理者権限が必要な場所）に置き、MDMで配布する想定です。

:::note warn
MCPなどの組織ガバナンスは、IAM Identity CenterかAPIキーで認証したユーザーにしか効きません。
Builder IDやソーシャルログインのユーザーは対象外です。
サインイン制限を端末に配らない限り、「個人アカウントでログインされたら統制が外れる」状態が残ります。
:::

### Enterprise Managementで細かい制御が追加された

2026年11月に、**Kiro Enterprise Management**がプレビューとして公開されました。
Kiro console の Controls で、KGLというYAML形式のポリシー（フォーム画面でも編集可）を管理します。
対象はモデル、MCP、シェル、ネットワークとWeb、ファイルシステム、Skills/Powers/エージェント、プライバシーとテレメトリーなどです。

管理者目線で便利な点は次のとおりです。

- **ユーザー・グループ単位のルール**を書ける（IdCや外部IdPのグループIDを指定）
- 公開（Publish）ごとに**リビジョン番号が付き、履歴を追える**。公開前に差分も確認できる
- ルールと設定は最大5分でクライアントに届く。**組織のsteeringファイルの配布**もできる
- プロンプトログのS3出力、ユーザー別の日次利用レポート（CSV/OpenTelemetry）、操作ごとのイベント送信を同じポリシーで設定できる

一方で、本番で頼るには注意点があります。

- プレビューで、**us-east-1とeu-central-1のプロファイルのみ**が対象
- 強制の主体が項目ごとに違う。モデル利用、APIキー作成、Web検索は**Kiroのサービス側で判定**されるが、MCP、シェル、ファイル操作は**クライアント側の強制**（Kiro CLI 3.1.0以降 / Kiro IDE 1.4.154以降が必要）
- MCPのdenyルールが一部ユーザーで効かない、といった挙動がドキュメント上で「調査中」とされている
- フィールド名を打ち間違えるとエラーにならず無視され、ルールが全体に効いてしまう

:::note warn
クライアント側で強制される制御は、端末の管理者権限を持つユーザーには迂回され得ると公式に明記されています。
これはKiroに限らず、ほかの4製品も同様です。
:::

### Kiroの評価

**AWS中心の組織にとっては、導入時の手間が少ない**製品です。
ID、課金、ログ出力先（S3）、暗号化キー（KMS）がすべて既存のAWSアカウント内に収まり、主要な制御はコンソールのトグルで済みます。

弱点は2つです。

- シェルやファイル操作の細かい制御はEnterprise Management（プレビュー）頼みで、リージョンも限られる
- デプロイオプションの選び方（IdCの組織インスタンスかアカウントインスタンスか）で効くガバナンスが変わり、後から変えにくい

## 観点別の比較表

ここまでの内容を観点ごとにまとめます。
「管理の入口」と「ID統制の土台」は、自社がどの基盤を持っているかで評価が逆転するため、記号を付けずに事実だけを並べています。

| 観点 | Antigravity | Claude Code | Codex | GitHub Copilot | Kiro |
|------|-------------|-------------|-------|----------------|------|
| 管理の入口 | Google Cloudコンソール（Gemini Enterprise） | claude.ai管理コンソール＋端末配布 | ChatGPT Admin Console（Agent security） | GitHubのAI controls＋`.github-private`リポジトリ | AWSアカウント内のKiro console |
| ID統制の土台 | Google Cloud IAM / WIF | Claudeの組織（SSO / SCIM） | ChatGPTワークスペース（SSO / SCIM / RBAC） | GitHub Enterprise / Organization | IAM Identity Center / 外部IdP |

以下は「その観点でガバナンスを効かせやすいか」の相対評価で、筆者の判断です。

| 観点 | Antigravity | Claude Code | Codex | GitHub Copilot | Kiro |
|------|-------------|-------------|-------|----------------|------|
| 個人アカウントの封じ込め | △ サインイン制限は見当たらず、ゲートウェイ構成が必要 | △ 端末側のログイン強制、またはテナント制限 | △ 端末の`requirements.toml`かMDMが必要 | ○ ネットワークで個人プランを遮断できる | △ 端末の`managed-settings.json`が必要 |
| 端末なしでの配布 | ○ コンソール設定 | ○ Server-managed（1時間ごと） | ○ Agent securityのクラウド配布 | ◎ リポジトリで管理し、PRでレビューできる | ○ コンソール設定、Enterprise Managementは5分で反映 |
| 制御の深さ | ○ ファイル・ターミナル・ブラウザ・MCP・モデル | ◎ 最も細かい | ○ 承認・サンドボックス・ネットワークまで対応 | ○ 権限・サンドボックス・プラグイン・MCP | ○ 主要項目はGA、シェル・ファイルはプレビュー |
| グループ単位の差分 | △ プロジェクト単位。グループ別設定は見当たらず | △ Enterpriseのみ、設定がマージされない | ◎ 優先度付きOverrides | ○ Enterprise teamsで上書き。複数Orgでは緩い側が適用 | ○ アカウント上書き＋（プレビュー）グループルール |
| 迂回されにくさ | △ 強制方式の記載が少なく、検証が必要 | △ 環境変数やAPIキーで取得自体を回避できる | ○ 端末MDMがクラウド設定より上位 | ○ MDMが最上位で、一部キーは厳しい側に合成 | ○ モデル・Web検索・APIキーはサービス側で判定 |
| 可視化・監査 | ◎ プロンプトログを自社プロジェクトのCloud Loggingに保存 | ○ OpenTelemetry、Analytics/Compliance API | ○ Analytics/Compliance API、OpenTelemetry | ○ 監査ログ（エージェントの操作）、usage metrics API | ◎ プロンプトログを自社アカウントのS3に保存 |
| リスク機能の初期値 | ○ MCP・外部モデルはオフ。ターミナル自動実行はオン | ○ 管理者が明示的に閉じる | ○ Codex cloudはEnterpriseで初期オフ | ◎ cloud agent・外部MCPは初期オフ | ◎ APIキー・Cloud Sessionsは初期オフ |

## どの組織でも共通してやるべきこと

製品を問わず、ガバナンスを実効性のあるものにするために必要なことは共通しています。

- **端末かネットワークの統制手段を用意する**。個人アカウントでの利用は、管理コンソールの設定だけでは封じられない
- **組織のID以外でのログインを封じる**。統制が効くのは組織のIDでログインしたセッションだけ
- **クライアント側強制の限界を前提にする**。端末の管理者権限の扱い、ネットワーク側の制御（プロキシやテナント制限）と組み合わせる
- **ローカルとクラウド実行を分けて確認する**。端末に配ったポリシーはクラウド実行には届かない。サーバー配布の設定がクラウド実行に効くかどうかも製品ごとに違う
- **設定後に、許可される操作と拒否される操作の両方を実機で確認する**。どの製品も「設定はしたが効いていない」ケースがドキュメントに書かれている

## まとめ

- **Antigravityは、Google Cloudを使っている組織なら設定が1画面に集まっていて始めやすい**。ただしグループ単位の差分と個人アカウントの封じ込めには工夫が要る
- **Claude Codeは最も深く統制できるが、設計量も多い**。MDMが整っている組織なら強力だが、Server-managed settingsだけに頼ると迂回経路が残る
- **Codexは、ChatGPT Enterpriseを使っている組織なら、グループ単位の配布が最も整理しやすい**。ただしローカル/クラウド/APIの境界ごとに確認が要る
- **GitHub Copilotは、GitHub Enterpriseを使っている組織なら、ポリシーをコードと同じように管理できる**。個人プランをネットワークで遮断できるのも強み。複数Organizationにまたがるユーザーの扱いには注意
- **Kiroは、AWS中心の組織なら既存のAWS資産の上で統制できる**。シェルやファイルの細かい制御はEnterprise Management（プレビュー）の成熟待ち
- どれを選んでも、**個人アカウントの封じ込めとクライアント側強制の限界には、端末管理やネットワーク制御で対応する必要がある**

「どれが一番ガバナンスを取りやすいか」という問いには、「すでに持っているID基盤と端末管理にいちばん近いもの」と答えるのが現実的だと考えています。
機能の一覧表だけで比べるのではなく、自社の基盤に乗せたときに、どこまでが設定だけで済み、どこから先に追加の仕組みが必要になるかを見積もってから選ぶことをおすすめします。
