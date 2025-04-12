---
tags:
  - MCP
  - Rails
---

[https://www.m3tech.blog/entry/future-with-mcp-servers:embed]
[https://zenn.dev/ubie_dev/articles/f927aaff02d618:embed]

最近 Web 開発業界で、「MCP サーバー」というものが注目されている。自社で管理している任意の情報システムと生成 AI ツールを連携できるような仕組みとして、活用の可能性が模索されているようだ。

簡単に実装できそうな様子があったので、まずはプライベートで実験してみたいと思って、自作の短歌投稿サイト Utakata と連携する MCP サーバーを作ってみた。

MCP サーバーの構築に興味はあるが、やり方がよく分からないような方の参考になるように、作り方を紹介してみる。

### 今回やったこと

- Utakata の Rails アプリケーションに、ユーザーごとの短歌の一覧を JSON の配列で返す API を作成する
- ローカル環境で動作する MCP サーバーを構築し、ユーザーごとの短歌の一覧を取得する Tool を作成する
- Claude Desktop に MCP サーバーを登録し、AI とのチャットでユーザーの投稿短歌についてやりとりできるようにする

### GitHub リポジトリ

[https://github.com/fuyu77/mcp-utakata:embed]

今回作ってみた MCP サーバーの実装を、動作確認できる形で GitHub に公開している。

### Utakata にユーザーごとの短歌一覧 API を作成

今回のユースケースでは、MCP サーバーから、連携するアプリケーションの API をリクエストして情報を取得する仕組みだ。

そのための API をまず用意する必要がある。以下のような JSON の配列を返す API を作ってみた。

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

Rails での API の実装方法の紹介は今回の本筋でないので省略するが、興味がある方は[GitHub に公開されている Utakata の実装](https://github.com/fuyu77/utakata/blob/master/app/controllers/api/posts_controller.rb)を参考にしてみて欲しい。

### MCP サーバーの実装

[https://github.com/modelcontextprotocol/typescript-sdk:embed]

公式の SDK が用意されているので、これを使って開発するのがお手軽だ。現状 5 種類の言語に対応されているようで、今回は typescript-sdk を用いて、Node.js 環境で動作するように実装する。

SDK の README で、Resources, Tools, Prompts という 3 つの概念が紹介されていて、使い分けの理解が難しい印象があるが、試行錯誤してみて、Claude Desktop と連携して使う用途では、Tool として作っておくのがお手軽に実行できて良さそうだった。

index.js のファイルを作成し、以下のように、 `fetch-user-tanka` の Tool を実装する。

[https://gist.github.com/fuyu77/61f267d038b66695cb6ca6afad4c4e44:embed#gist61f267d038b66695cb6ca6afad4c4e44]

Node.js の fetch で Utakata の API から情報取得し、JSON の配列を整形して以下のようなテキストをアウトプットする仕組みだ。

```markdown
# ユーザー ID: 5 の短歌一覧（最新順）

- 片耳にマスクをかけて池の面をながれる風に呼応している（投稿日時: 2021-05-15T17:39:00.000+09:00、いいね数: 32）
- 人びとの残りをもとめ散る花の上を歩いてゆく鳩の群れ（投稿日時: 2021-03-29T13:45:16.888+09:00、いいね数: 17）
```

### MCP サーバーの動作確認

[MCP Inspector](https://github.com/modelcontextprotocol/inspector)という便利なツールが公式で用意されていて、ローカルで動作確認できる。

```
npx @modelcontextprotocol/inspector node index.js
```

[f:id:fuyu77:20250406135726p:plain]

### Claude Desktop との連携

MCP サーバーの動作確認ができたら、あとは生成 AI ツールと連携して使ってみるだけだ。MCP サーバーとの連携に対応している任意のツールと連携可能だが、今回は[Claude Desktop](https://claude.ai/download)と連携して使ってみる。この記事では Mac の環境で確認した結果を紹介する。

[https://claude.ai/download:embed]

まずは、Claude のデスクトップアプリケーションをインストールする。

[https://modelcontextprotocol.io/quickstart/user:embed]

公式ドキュメントに記載の方法を参考に連携設定する。Mac の場合、Claude のアプリケーション内の設定ではなく、Mac のヘッダーメニューの「Claude > Settings > Developer」から該当メニューに辿り着く必要がある。「Get Started」ボタンを押すと設定ファイルが `~/Library/Application Support/Claude/claude_desktop_config.json` のように作成されるので、エディタで編集して MCP サーバーとの連携設定を追加する。

```json
{
  "mcpServers": {
    "utakataTankaReader": {
      "command": "/Users/fuyu77/.nodenv/shims/node",
      "args": ["/Users/fuyu77/mcp-utakata/index.js"]
    }
  }
}
```

私の環境では、上のような設定で動作した。node コマンドと js ファイルのパスは、個別の環境の値に置き換える必要がある。

この設定を保存して、Claude のアプリケーションを起動すると、Utakata の情報を参照した AI とのやりとりが可能になった。

[f:id:fuyu77:20250406142102p:plain]
[f:id:fuyu77:20250406142320p:plain]

### TypeScript での実装

[https://modelcontextprotocol.io/quickstart/server:embed]

説明を簡単にするために、この記事では JavaScript での実装で紹介したが、TypeScript で実装してビルドする方法が上の公式ドキュメントに解説されている。

### まとめ

MCP サーバーの実装はこのように簡単にできて、任意の情報システムと生成 AI ツールとの連携が可能になるので、業界で大いに注目されているのも頷ける印象だ。

Utakata の投稿短歌についても、この仕組みを活用して何か面白いことができるかも知れない。もし Utakata の MCP サーバーの活用について何かアイディアがある方がいれば、[連絡先](https://utakatanka.jp/about#contacts)に記載の X アカウントの DM や、メールアドレスに連絡いただきたい。
