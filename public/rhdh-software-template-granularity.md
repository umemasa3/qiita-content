---
title: "RHDHのSoftware Templateはどの粒度で作るべきか、そして既存モノレポへどう差し込むか"
tags:
  - "Backstage"
  - "RedHatDeveloperHub"
  - "プラットフォームエンジニアリング"
  - "IDP"
  - "Scaffolder"
private: false
updated_at: ""
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

RHDH（Red Hat Developer Hub）やBackstageをIDPとして運用するとき、Software Template（Scaffolder）の作り方には設計判断が必要です。特に悩ましいのが**テンプレートの粒度**です。

一つ一つを細かいコンポーネント（ビルディングブロック）として作り、組み合わせて使うのか。それとも、ある程度まとまった複合ブロックとして展開するのか。どちらにもメリット・デメリットがあります。

さらに、細かいビルディングブロックとして作ったものを、新規リポジトリではなく**既存のモノレポにマージ**したいケースもあります。この場合、通常の「新規リポジトリ作成」とは違うやり方が必要になります。

この記事では、Software Templateの構成要素を整理した上で、粒度の設計判断と、既存モノレポへの差し込み方を検討します。

参考: [Backstage Docs - Writing Templates](https://backstage.io/docs/features/software-templates/writing-templates/)
/ [Builtin actions](https://backstage.io/docs/features/software-templates/builtin-actions/)

## Software Templateの構成要素

まず前提として、テンプレートの基本構造を押さえます。テンプレートは `template.yaml` で定義され、大きく2つのパートに分かれます。

- **parameters** — フロントエンドに表示されるフォーム入力。複数のステップ（ページ）に分割でき、`RepoUrlPicker` などの専用フィールドも使える
- **steps** — バックエンドで直列実行されるアクションのリスト

stepsで使う主なビルトインアクションは以下です。

| アクション | 役割 |
|-----------|------|
| `fetch:template` | skeletonを取り込み、変数を展開してファイル生成 |
| `fetch:plain` | テンプレート展開なしでファイルをコピー |
| `publish:github` | 新規リポジトリを作成して公開 |
| `publish:github:pull-request` | 既存リポジトリにPRを作成 |
| `catalog:register` | 生成物をSoftware Catalogに登録 |

この「stepsが直列のアクションの並び」という構造が、粒度設計と合成の鍵になります。

## 粒度の2つのアプローチ

テンプレートの粒度は、大きく2つのアプローチに分けられます。

![ビルディングブロック型とまとめたブロック型の比較](https://github.com/umemasa3/qiita-content/blob/main/public/images/rhdh-template-granularity.drawio.svg?raw=true)

### アプローチA：ビルディングブロック型（細かく分ける）

「サービスの雛形」「CI設定」「監視設定」「IaC」などを、それぞれ独立した小さなテンプレート（またはskeleton）として作り、組み合わせて使うスタイルです。

**メリット：**

- 再利用性が高い。1つのブロックを複数のテンプレートで使い回せる
- 変更の影響範囲が小さい。CI設定だけ直せば、それを使う全テンプレートに反映しやすい
- 責任分界が明確。各ブロックを別チームが保守できる

**デメリット：**

- 組み合わせの管理が複雑になる。どのブロックをどう組むかの設計が要る
- 利用者が「何と何を組み合わせればいいか」を判断する負荷が生まれることがある
- テンプレート数が増え、カタログが煩雑になりやすい

### アプローチB：まとめたブロック型（複合テンプレート）

「Webサービス一式（雛形＋CI＋監視＋IaC）」のように、必要なものをまとめて1つのテンプレートで展開するスタイルです。

**メリット：**

- 利用者は1つ選ぶだけで完結する。判断負荷が低い
- 生成物の一貫性を保ちやすい。組み合わせのズレが起きない
- カタログがシンプルになる

**デメリット：**

- 再利用性が低い。似たテンプレートで同じ内容を重複して持ちがち
- 変更が波及しにくい。共通部分を直しても各テンプレートに手で反映が要る
- 1つのテンプレートが肥大化し、保守が重くなる

### どちらを選ぶか

この2つは排他ではなく、**内部はビルディングブロックで持ち、利用者にはまとめたブロックとして見せる**のが現実的な落としどころです。Scaffolderの `fetch:template` は複数ステップ重ねられるので、これが可能になります。

## ビルディングブロックをどう合成するか

`fetch:template` は1つのテンプレート内で複数回呼べます。これを使って、細かいskeletonを重ねて1つの成果物を組み立てられます。

```yaml
steps:
  # サービス雛形を取り込む
  - id: fetchBase
    name: Fetch Base
    action: fetch:template
    input:
      url: ./skeletons/service
      values:
        name: ${{ parameters.name }}

  # CI設定を別skeletonから重ねる
  - id: fetchCI
    name: Fetch CI
    action: fetch:template
    input:
      url: ./skeletons/ci
      targetPath: ./.github
      values:
        name: ${{ parameters.name }}
```

ポイントは以下です。

- `targetPath` で、取り込んだファイルの配置先ディレクトリを指定できる。ブロックごとに置き場所を分けられる
- `url` に絶対URLを指定すれば、**別リポジトリで管理しているskeletonも取り込める**。共通ブロックを中央リポジトリに置いて共有できる
- `if` で条件付き実行、`each` で繰り返し実行ができる。「CIを有効にする場合だけCIブロックを取り込む」といった分岐が書ける

:::note info
この仕組みを使えば、利用者に見せるテンプレートは1つ（まとめたブロック型のUX）にしつつ、中身は再利用可能なskeletonの組み合わせ（ビルディングブロック型の保守性）にできます。粒度の2択は、合成で両取りできるということです。
:::

## 既存モノレポへどう差し込むか

ビルディングブロックとして細かく作ったものを、新規リポジトリではなく**既存のモノレポにマージ**したい、というのが次の論点です。

通常のテンプレートは `publish:github`（または `publish:gitlab`）で新規リポジトリを作成します。
しかし既存リポジトリに差し込む場合は、**`publish:github:pull-request`**（GitLabなら **`publish:gitlab:merge-request`**）を使います。
これは新規リポジトリを作らず、既存リポジトリにPR/MRを作成するアクションです。

![既存モノレポへの差し込みフロー](https://github.com/umemasa3/qiita-content/blob/main/public/images/rhdh-monorepo-merge.drawio.svg?raw=true)

### 差し込みの基本形

```yaml
steps:
  # 生成物をモノレポ内の配置先パスに展開する
  - id: fetch
    name: Fetch Skeleton
    action: fetch:template
    input:
      url: ./skeletons/service
      targetPath: packages/${{ parameters.name }}
      values:
        name: ${{ parameters.name }}

  # 既存リポジトリにPRを作成する（新規リポジトリは作らない）
  - id: publish
    name: Create Pull Request
    action: publish:github:pull-request
    input:
      repoUrl: ${{ parameters.repoUrl }}
      branchName: add-${{ parameters.name }}
      title: "Add ${{ parameters.name }} to monorepo"
      description: "Generated from Backstage template"
```

ポイントは以下です。

- `fetch:template` の `targetPath` で、モノレポ内のどのディレクトリ（例: `packages/<name>`）に差し込むかを指定する
- `publish:github:pull-request` は、対象リポジトリに新しいブランチを切ってPRを作る。既存コードを直接壊さず、レビューを挟める
- 前回のガバナンス記事で触れた「承認を挟む」フローとも相性がよい。PRレビューが承認ゲートになる

:::note info
GitLabでも同じことができます。`publish:gitlab:merge-request` アクションが用意されていて、新しいブランチを切ってマージリクエスト（MR）を作成します。
`sourcePath` / `targetPath` で差し込み先を指定でき、`removeSourceBranch` などのオプションもあります。
GitHubの `publish:github:pull-request` とほぼ同じ考え方で、モノレポへの差し込みに使えます。
:::

### 新規リポジトリと既存モノレポ、テンプレートは分けるべきか

「新規リポジトリ作成用」と「既存モノレポ差し込み用」で、テンプレートを2つ用意すべきか——これは実際に迷うところです。
結論としては、**1つのテンプレートでも両対応できます**。`parameters` に出力先の種別を持たせ、`if` 条件で publish アクションを出し分ければよいだけです。

```yaml
parameters:
  - title: Destination
    properties:
      destination:
        title: 出力先
        type: string
        enum: ["new-repo", "existing-monorepo"]
        default: "new-repo"

steps:
  - id: fetch
    action: fetch:template
    input:
      url: ./skeletons/service
      # 既存モノレポのときだけサブディレクトリに差し込む
      targetPath: ${{ parameters.destination === "existing-monorepo" and "packages/" + parameters.name or "." }}
      values:
        name: ${{ parameters.name }}

  # 新規リポジトリ
  - id: publishNew
    action: publish:github
    if: ${{ parameters.destination === "new-repo" }}
    input:
      repoUrl: ${{ parameters.repoUrl }}

  # 既存モノレポにPR
  - id: publishPR
    action: publish:github:pull-request
    if: ${{ parameters.destination === "existing-monorepo" }}
    input:
      repoUrl: ${{ parameters.repoUrl }}
      branchName: add-${{ parameters.name }}
      title: "Add ${{ parameters.name }} to monorepo"
```

ただしトレードオフがあります。

- **1テンプレートに統合** — 利用者が選ぶテンプレートが1つで済む。ただし `if` 分岐が増え、テンプレートが読みにくくなる。分岐が2〜3個までなら許容範囲
- **2テンプレートに分割** — それぞれが単純で読みやすい。ただし共通のskeletonやparametersを二重管理しがち（skeletonを共有すれば緩和できる）

判断の目安は、**分岐の数と、利用者にとっての分かりやすさ**です。出力先の違いだけなら `if` ひとつで済むので1テンプレートで十分。もし新規と既存で生成物の構成まで大きく変わるなら、分けた方が読みやすくなります。

### 差し込み時に気をつけること

:::note warn
既存モノレポへの差し込みでは、`targetPath` の設計が重要です。配置先が既存ファイルと衝突すると、意図しない上書きや不整合が起きます。
差し込むのは「新しいディレクトリ配下」に限定し、ルートの共通ファイル（ルートの `package.json`、CI設定、`catalog-info.yaml` 等）を上書きしない設計にするのが安全です。
:::

- **配置先の衝突を避ける** — モノレポの新しいサブディレクトリに閉じて差し込む。既存の共通ファイルには触れない
- **カタログ登録** — 既存リポジトリに `catalog-info.yaml` を追加するなら `catalog:register` の `repoContentsUrl` で登録する。モノレポは1リポジトリに複数コンポーネントが載るので置き場所を決めておく
- **失敗時のクリーンアップ** — Scaffolderには `if: ${{ failure() }}` で失敗時だけ走るステップや、`if: ${{ always() }}` で常に走るステップを書ける。途中失敗で中途半端なPRやブランチが残らないよう、後始末のステップを用意しておく
- **モノレポ側の規約に合わせる** — 差し込む内容が、既存モノレポのディレクトリ規約・ビルド設定・lint設定に沿っているか。ここが崩れると、差し込んだ瞬間にCIが壊れる

## IaC（Terraform等）を扱う場合の責任分界

差し込む内容がアプリのコードではなく、**AWSリソースを定義するTerraform**のようなIaCだと、話が一段複雑になります。「誰がその変更に責任を持つのか」を決めておく必要があるからです。

### Terraformでも既存リポジトリへのPRは推奨できる

結論から言うと、IaCこそPR方式が向いています。

- PR上で `terraform plan` の結果を確認できる。いきなりapplyせず、何が変わるかを見てから進められる
- 変更履歴がGitに残り、監査とロールバックがしやすい
- 「申請 → PR生成 → plan確認 → 承認 → apply」という流れは、承認付きセルフサービスそのもの

直接applyや自動マージより、PR/MRを挟む方がミッションクリティカルな環境では安全です。

### 責任は「レール」と「その上を走る判断」で分ける

「実行責任はプラットフォームチームか、アプリチームか」という問いには、何に対する責任かで分けて考えるのが実務的です。

- **モジュール・テンプレート・ガードレール（どう作るかの規格）** → プラットフォームチームの責任。Terraformモジュール、Scaffolderテンプレート、許可するリソース種別・命名規約・タグ強制・Policy as Codeを提供し保守する
- **その変更を出す判断と、生成する値（どのリソースをいくつ作るか）** → アプリチームの責任。自分たちのニーズでパラメータを入力し、PRを出し、中身を理解して進める

プラットフォームチームが「レール」を敷き、アプリチームがその上を走る判断をする、という分け方です。プラットフォームチームがapplyまで全部代行すると、プラットフォームチームがボトルネックになり、かつアプリチームが自分の作るものを理解しないまま増えていきます。

:::note info
承認ゲート（誰がマージ権を持つか）は、実行責任とは別に設計できます。共通基盤側のミッションクリティカルなリソースなら、「アプリチームがPRを出す（実行の判断と責任）」「共通基盤チームがマージを承認する（=applyのゲート）」という二段構えが現実的です。これは『共通基盤側が申請パラメータをチェックして承認したら自動実行する』という設計とも一致します。
:::

CODEOWNERSやブランチ保護で、IaCのディレクトリは共通基盤チームの承認を必須にする、といった形でこのゲートを技術的に担保できます。

## 判断のまとめ

粒度と差し込みの設計判断を整理します。

- **粒度は「内部はブロック、外はまとまり」が基本** — `fetch:template` の重ねがけで、保守性（ビルディングブロック）と利用者体験（まとめたブロック）を両取りする
- **共通ブロックは中央リポジトリに置く** — 絶対URLで取り込めるので、CI設定やセキュリティ設定など全社共通のskeletonは一箇所で管理する
- **新規と既存でpublishアクションを使い分ける** — 新規は `publish:github`、既存モノレポは `publish:github:pull-request`（GitLabは `publish:gitlab:merge-request`）
- **1テンプレートで両対応もできる** — `parameters` の出力先種別と `if` 分岐で、新規/既存を1つのテンプレートにまとめられる
- **既存モノレポへの差し込みはサブディレクトリに閉じる** — `targetPath` で新しいディレクトリに限定し、共通ファイルを上書きしない
- **PRを承認ゲートにする** — `publish:github:pull-request` のPRレビューが、そのまま統制のポイントになる

## まとめ

Software Templateの粒度は「細かいビルディングブロック」か「まとめた複合ブロック」かの二択に見えますが、Scaffolderの `fetch:template` を重ねる合成の仕組みを使えば、内部は再利用可能なブロック、利用者には1つのまとまり、という形で両取りできます。

既存モノレポへの差し込みは、新規リポジトリ作成とは別で、`publish:github:pull-request` を使って既存リポジトリにPRを作る形になります。`targetPath` で差し込み先を新しいサブディレクトリに限定し、共通ファイルを壊さないこと、そしてPRレビューを承認ゲートとして活かすことが、安全に運用する鍵です。

テンプレートを「どう作るか」だけでなく「どこに、どう届けるか」まで設計に含めることが、モノレポ主体の組織でScaffolderを実運用に乗せるポイントだと考えています。そしてTerraformのようなIaCを扱うなら、「プラットフォームがレールを敷き、アプリチームがその上を走る判断をし、承認ゲートは共通基盤が持つ」という責任分界まで一緒に設計しておくことが、安全な自動化につながると考えています。
