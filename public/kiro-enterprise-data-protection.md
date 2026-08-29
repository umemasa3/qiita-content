---
title: "エンタープライズでKiroを使うなら、ログイン方法でデータの扱いが変わる"
tags:
  - "Kiro"
  - "AI駆動開発"
  - "セキュリティ"
  - "エンタープライズ"
  - "IAMIdentityCenter"
private: false
updated_at: ""
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

私は普段からAIコーディングエージェントのKiroをメインで使っています。個人で使う分には気軽にサインインして使い始めればいいのですが、会社の業務でKiroを使うとなると、一つ見落としやすい落とし穴があります。

**どうやってログインするかで、入力したコードやプロンプトがモデルの学習に使われるかどうかが変わる**という点です。

GitHubアカウントやGoogleアカウントで気軽にログインして、有料プランを契約して使っていたとしても、それはKiroの分類上「individual subscriber（個人契約者）」であって、エンタープライズユーザーではありません。そして個人契約者のデータは、サービス改善のために利用される可能性があります。

この記事では、Kiro公式のデータ保護ドキュメントをもとに、エンタープライズ企業でKiroを使う場合に何に気をつけるべきかを整理します。

参考: [Kiro Docs - Data protection](https://kiro.dev/docs/privacy-and-security/data-protection/)

## individual subscriberとenterprise userの違い

Kiroのユーザーは大きく分けて2つの区分があります。

- **individual subscriber（個人契約者）** — ソーシャルログイン（GitHub、Google）またはAWS Builder ID経由でアクセスするユーザー。有料プランを契約していてもこの区分に入る
- **enterprise user（エンタープライズユーザー）** — AWSコンソールから追加・サブスクライブされ、IAM Identity Center、Okta、Microsoft Entra ID経由でアクセスするユーザー

ここで重要なのは、**「有料かどうか」ではなく「どの認証経路でログインしているか」で区分が決まる**ということです。会社のお金で有料プランを契約していても、GitHubアカウントでログインしていればそれはindividual subscriberになります。

## サービス改善への利用：individualは対象、enterpriseは対象外

Kiroは、サービス改善のために以下のようなコンテンツを利用する場合があるとしています。

- ユーザーがKiroに投げた質問
- その他の入力
- Kiroが生成した応答やコード

用途としては、よくある質問への応答改善、運用上の問題の修正、デバッグ、そして**モデルのトレーニング**が挙げられています。

このサービス改善への利用について、区分ごとの扱いは明確に分かれています。

- **individual subscriber と Free Tier** — サービス改善の対象になる（デフォルトで有効）
- **enterprise user** — サービス改善には利用されない。AWSによって自動的にテレメトリとコンテンツ収集からオプトアウトされている

つまり、業務で書いたコードやプロンプトをモデル学習に使われたくないのであれば、enterprise userとして使う必要があります。

なお、Amazon Q Developer Proのサブスクリプションを持っていて、そのAWSアカウント経由でKiroにアクセスしている場合も、サービス改善には利用されないとされています。

## データの保存リージョンも変わる

サービス改善だけでなく、データがどこに保存されるかも区分によって変わります。

- **Free Tier / individual subscriber** — プロンプトや応答などのコンテンツは米国東部（バージニア北部）リージョンに保存される
- **enterprise user** — コンテンツはKiroプロファイルが設定されているリージョンに保存される（サービス提供・維持のため）

日本のエンタープライズ企業でデータの保存場所に制約がある場合、この違いは無視できません。enterprise userであればプロファイルのリージョンを選べる余地があります。

## enterprise userになるには：IAM Identity Center等での認証が必須

ここが本題です。enterprise userになるには、個人でサインインするのではなく、組織のAWSアカウント側でセットアップが必要になります。

Kiroのドキュメントでは、enterprise userを次のように定義しています。

> AWSコンソールを通じてKiroサブスクリプションに追加・登録されたユーザーで、
> IAM Identity Center、Okta、Microsoft Entra ID経由でKiroにアクセスできるユーザー。

（[Kiro Docs - Concepts](https://kiro.dev/docs/enterprise/concepts/) より要約）

要点は以下の通りです。

1. 管理者がAWSコンソール（Kiro console）でKiroサブスクリプションを作成する
2. IAM Identity Center、Okta、Microsoft Entra IDのいずれかでIDプロバイダを接続する
3. ユーザーまたはグループをサブスクリプションに追加する
4. ユーザーはそのIDプロバイダ経由でKiroにログインする

この経路でログインして初めて、そのユーザーはenterprise userとして扱われます。逆に言えば、いくら会社が契約していても、開発者が個人のGitHubアカウントでログインしていれば、その利用はenterprise扱いになりません。

### ログイン経路による扱いの違い

![Kiroのログイン経路とデータの扱いの違い](https://github.com/umemasa3/qiita-content/blob/main/public/images/kiro-login-data-handling.drawio.svg?raw=true)

## その他のデータ保護

区分に関わらず、Kiroには以下のデータ保護があります。

- **転送時の暗号化** — 顧客とKiro間、Kiroと下流依存先の間の通信はすべてTLS 1.2以上で保護される
- **保管時の暗号化** — AWS KMSのAWS所有キーでデータを暗号化。enterpriseの管理者はカスタマーマネージドキー（自分で作成・所有・管理するKMSキー）を使う選択肢もある

カスタマーマネージドキーを使えば、KMSキーへのアクセスを制御することで、データへのアクセスを直接コントロールできます。データの機密性が高い組織では検討する価値があります。

## エンタープライズ導入時のチェックポイント

以上を踏まえて、会社でKiroを導入・展開する際に確認すべき点をまとめます。

- 開発者が**個人のソーシャルログインで使い始めていないか**を確認する
- IAM Identity Center（またはOkta、Microsoft Entra ID）でIDプロバイダを接続し、AWSコンソールからサブスクリプションを管理する
- Kiroプロファイルのリージョンを、データ保存要件に合わせて設定する
- 機密性が高い場合はカスタマーマネージドキーの利用を検討する
- 「有料契約している＝安全」ではないことを、利用者に周知する

## まとめ

KiroのようなAIコーディングエージェントは、業務コードやプロンプトという極めて機密性の高い情報を扱います。その情報がモデル学習に使われるかどうかは、契約の有無ではなく**ログイン経路**で決まります。

- ソーシャルログインやAWS Builder IDはindividual subscriber扱いで、サービス改善（モデル学習を含む）の対象になる
- enterprise userはIAM Identity Center等の経由が必須で、サービス改善から自動的に除外される
- データの保存リージョンもenterpriseなら選べる余地がある

会社でKiroを展開するなら、「まず正しい認証経路でログインさせる」ことが最初の一歩になります。便利さだけで個人アカウントで使い始めてしまうと、知らないうちに業務コードが学習データに流れてしまう可能性があります。導入の最初の設計でここを外さないようにしたいところです。
