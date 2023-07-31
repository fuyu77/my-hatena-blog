---
tag: Rails
---
[[20180624短歌投稿サイトUtakataをリリースしました]]

[https://utakatanka.jp/:embed]

Google Analyticsで2023年7月1日からUAの計測ができなくなり、GA4への移行が必須になるということで、個人で開発している短歌投稿サイトUtakataもGA4に移行したのだけれど、移行した途端にページビューの値が極端に減少してしまうという事態に遭遇した。結論から言うと、RailsのTurbo利用のサイトで正しく計測するために、UAのときとは異なるイベント送信が必要という原因だったのだけれど、関連ワードでググっても（Turboの利用者数が少なすぎるのか）解決記事の類が出てこなかったので、この問題の解決方法を書いておく。

### UA時代のTurbo対策

UA時代からTurbo利用でページビューを正しく測定するには一工夫必要だった。

[https://gist.github.com/fuyu77/1b6ed79a1597e18403dd7e7fe4901a7b:embed#gist1b6ed79a1597e18403dd7e7fe4901a7b]

[https://gist.github.com/fuyu77/8ff1ed69d040429ec66466b09a0c0938:embed#gist8ff1ed69d040429ec66466b09a0c0938]


上のような形で<code>turbo:load</code>をeventをトリガーにgtag関数を実行しないと正しくページビューが計測されない。

### GA4でのTurbo対策

GA4移行に際して、上のUAタグをただGタグに置き換えただけだと、ページビューが少なく計測される問題が発生した。

[https://developers.google.com/analytics/devguides/collection/ga4/page-view?hl=ja:embed]

[https://gist.github.com/fuyu77/2b56aa8187dd8a833817025311f1ff86:embed#gist2b56aa8187dd8a833817025311f1ff86]

GA4の開発ドキュメントに記載のある、ページビューイベントを別に送信する処理を追加するとこの問題は解決された。

### 余談 ―UA時代のUtakataのページビュー振り返り―

[f:id:fuyu77:20230718213533p:plain]

UAで記録していた2019/01/14〜2023/06/30までのUtakataのページビューの遷移の上のようになっている。今年に入ってから猛烈に伸びていて、最近では毎日4000前後のPVがあるところまで成長した。

[https://www.utayom.in/pages/utayomin_release_202301:embed]

この伸びは、最も人気があった短歌投稿サイト「うたよみん」のサービス終了による代替サービスとしての需要が大きい。

とは言え、Utakataもまた、多くのユーザーに支持され、日々使っていただけていることも事実だ。

[https://fuyu.hatenablog.com/entry/2018/06/24/233652:embed]

[https://github.com/fuyu77/utakata:embed]

Utakataは新卒で入った会社を逃げるように辞めて、Webエンジニアになるための勉強をしていた頃にリリースしたサービスで、コツコツと開発を続けて言語やライブラリのバージョンも最新にアップデートしているので、最初は滅茶苦茶なRailsアプリだったのが、今ではHotwire（Turbo + Stimulus）利用のRails 7のアプリケーションとしてそれなりに真っ当な実装になっていると思う。コツコツと継続していることに結果がついてくるのはやはり嬉しいものだ。