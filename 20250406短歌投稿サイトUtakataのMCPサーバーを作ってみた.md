---
tags:
  - MCP
  - Rails
---

[https://www.m3tech.blog/entry/future-with-mcp-servers:embed]
[https://zenn.dev/ubie_dev/articles/f927aaff02d618:embed]

最近Web開発業界で、「MCPサーバー」というものが注目されている。自社で管理している任意の情報システムと生成AIツールを連携できるような仕組みとして、活用の可能性が模索されているようだ。

簡単に実装できそうな様子があったので、まずはプライベートで実験してみたいと思って、自作の短歌投稿サイトUtakataと連携するMCPサーバーを作ってみた。

MCPサーバーの構築に興味はあるが、やり方がよく分からないような方の参考になるように、作り方を紹介してみる。

### 今回やったこと

- UtakataのRailsアプリケーションに、ユーザーごとの短歌の一覧をJSONの配列で返すAPIを作成する
- ローカル環境で動作するMCPサーバーを構築し、ユーザーごとの短歌の一覧を取得するToolを作成する
- Claude DesktopにMCPサーバーを登録し、AIとのチャットでユーザーの投稿短歌についてやりとりできるようにする

### GitHubリポジトリ

[https://github.com/fuyu77/mcp-utakata:embed]

完成品を動作確認できる形でGitHubに公開している。

### Utakataにユーザーごとの短歌一覧APIを作成

今回のユースケースでは、MCPサーバーから、連携するアプリケーションのAPIをリクエストして情報を取得する仕組みだ。

そのためのAPIをまず用意する必要がある。以下のようなJSONの配列を返すAPIを作ってみた。

GET /api/users/:user_id/posts

```json
[
  {
    "id": 25401,
    "published_at": "2021-05-15T17:39:00.000+09:00",
    "tanka_text": "片耳にマスクをかけて池の面をながれる風に呼応している",
    "likes_count": 32
  },
  {
    "id": 23563,
    "published_at": "2021-03-29T13:45:16.888+09:00",
    "tanka_text": "人びとの残りをもとめ散る花の上を歩いてゆく鳩の群れ",
    "likes_count": 17
  }
]
```

RailsでのAPIの実装方法の紹介は今回の本筋でないので省略するが、興味がある方は[GitHubに公開されているUtakataの実装](https://github.com/fuyu77/utakata/blob/master/app/controllers/api/posts_controller.rb)を参考にしてみて欲しい。

### MCPサーバーの実装

[https://github.com/modelcontextprotocol/typescript-sdk:embed]

公式のSDKが用意されているので、これを使って開発するのがお手軽だ。現状5種類の言語に対応されているようで、今回はtypescript-sdkを用いて、Node.js環境で動作するように実装する。

SDKのREADMEで、Resources, Tools, Promptsという3つの概念が紹介されていて、使い分けの理解が難しい印象があるが、試行錯誤してみて、Claude Desktopと連携して使う用途では、Toolとして作っておくのがお手軽に実行できて良さそうだった。

build/index.jsのファイルを作成し、以下のように、 `fetch-user-tanka` のToolを実装する。

[https://gist.github.com/fuyu77/61f267d038b66695cb6ca6afad4c4e44:embed#gist61f267d038b66695cb6ca6afad4c4e44]

Node.jsのfetchでUtakataのAPIから情報取得し、JSONの配列を整形して以下のようなテキストをアウトプットする仕組みだ。

```markdown
# ユーザーID: 5 の短歌一覧（最新順）

- 片耳にマスクをかけて池の面をながれる風に呼応している（投稿日時: 2021-05-15T17:39:00.000+09:00、いいね数: 32）
- 人びとの残りをもとめ散る花の上を歩いてゆく鳩の群れ（投稿日時: 2021-03-29T13:45:16.888+09:00、いいね数: 17）
```

### MCPサーバーの動作確認

[MCP Inspector](https://github.com/modelcontextprotocol/inspector)という便利なツールが公式で用意されていて、ローカルで動作確認できる。

```
npx @modelcontextprotocol/inspector node build/index.js
```

[f:id:fuyu77:20250406135726p:plain]

### Claude Desktop

MCPサーバーの動作確認ができたら、あとは生成AIツールと連携して使ってみるだけだ。MCPサーバーとの連携に対応している任意のツールと連携可能だが、今回は[Claude Desktop](https://claude.ai/download)と連携して使ってみる。この記事ではMacの環境で確認した結果を紹介する。

[https://claude.ai/download:embed]

まずは、Claudeのデスクトップアプリケーションをインストールする。

[https://modelcontextprotocol.io/quickstart/user:embed]

公式ドキュメントの方法を参考に連携設定する。Macの場合、Claudeのアプリケーション内の設定ではなく、Macのヘッダーメニューの「Claude > Settings > Developer」から該当メニューに辿り着く必要がある。「Get Started」ボタンを押すと設定ファイルが `~/Library/Application Support/Claude/claude_desktop_config.json` のように作成されるので、エディタで編集してMCPサーバーとの連携設定を追加する。

```json
{
  "mcpServers": {
    "utakataTankaReader": {
      "command": "/Users/fuyu77/.nodenv/shims/node",
      "args": [
        "/Users/fuyu77/mcp-utakata/build/index.js"
      ]
    }
  }
}
```

私の環境では、上のような設定で動作した。nodeコマンドとjsファイルのパスは、個別の環境の値に置き換える必要がある。

この設定を保存して、Claudeのアプリケーションを起動すると、Utakataの情報を参照したAIとのやりとりが可能になった。

[f:id:fuyu77:20250406142102p:plain]
[f:id:fuyu77:20250406142320p:plain]

### まとめ

MCPサーバーの実装はこのように簡単にできて、任意の情報システムと生成AIツールとの連携が可能になるので、業界で大いに注目されているのも頷ける印象だ。

Utakataの投稿短歌についても、この仕組みを活用して何か面白いことができるかも知れない。もしUtakataのMCPサーバーの活用について何かアイディアがある方がいれば、[連絡先](https://utakatanka.jp/about#contacts)に記載のXアカウントのDMや、メールアドレスに連絡いただきたい。
