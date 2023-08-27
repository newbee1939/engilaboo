・RSC(React Server Components)って最近よく聞くけど、どんなものなのか分かっていない。。
・RSC と App Router ってどういう関係なんだろう？
・そろそろキャッチアップしとかないとマズイよな。。

今回はこういった声にお答えします。
なるべく丁寧に解説するので、少しでも参考になれば幸いです。

※全ての資料を読んで完璧にする
※ChatGPT で誤字脱字の修正
※ブログと Qiita に同じ内容投稿すればいいやん（ちょっと変えてもいいし）
※網羅性を高める
※ユーザー視点。何が知りたいか？ISR とか必要？

## React Server Components と App Router を今度こそちゃんと理解する

いきなり RSC の深いところから話しても難しいと思うので、順を追って説明していきます。

まずは React と Next.js についてです。

### React と Next.js について

React とは、UI を簡単に構築するための JavaScript ライブラリです。

「コンポーネント」という概念を使って宣言的に UI を定義することで、簡単に画面の構築を行うことができます。

そして、Next.js とは React のフレームワークです。

React の機能をさらに拡張してより使いやすくしたものと捉えて良いでしょう。

### React のレンダリング方式について

React は、Create React App で作成された状態では、CSR（クライアントサイドレンダリング）でレンダリングをします。

CSR とは、ブラウザ上で JavaScript を実行して DOM を生成しコンテンツを表示させる方法です。

ページの初期ロード時にはコンテンツは何も表示されず（白い画面）、ブラウザでの JavaScript の実行後に画面が表示されます。

React でのレンダリングは、以下のステップで行われます。

参考: https://react.dev/learn/render-and-commit#epilogue-browser-paint

1. Triggering a render

レンダリングのきっかけとなるトリガーを検知します。

ここでのトリガーとは、以下の二つです。

- コンポーネントの初期レンダリング
- コンポーネントの状態（state）の更新

このいずれかのトリガーを検知したとき、React はレンダリングを開始します。

2. Rendering the component

次に、実際にレンダリングを行います。

React における「レンダリング」は主に以下の３つを行います。

- 1. 対象のコンポーネントの呼び出し
- 2. 以前のコンポーネントの状態との比較
- 3. コミット（ブラウザレンダリング）する内容の決定

1 の対象のコンポーネントについて、最初のレンダリングでは、ルートコンポーネントを呼び出し、それ以降のレンダリングでは、状態更新がレンダリングのトリガーとなった関数コンポーネントを呼び出します。

そして 2 で以前のコンポーネントの状態との差分を計算し、（そもそも画面を更新するか否かも含めて）画面更新の内容を決定します。

そして、差分が検知されて画面を更新する必要がある場合、次の手順 3 を実行します。

つまり、React のレンダリングとは、「レンダリング対象のコンポーネントを呼び出して前回の内容との差分を比較し、何をブラウザレンダリング（Commit）をするかを決定すること」と言えるでしょう。

同じ「レンダリング」という名前が付いているので混同しがちですが、「ブラウザレンダリング（画面への描画）」とは違う概念なのでしっかりと区別するようにしましょう。

順番としては、「React のレンダリング」をした後に「ブラウザのレンダリング」を行なっているイメージです。

3. Committing to the DOM

React は 2 で以前の状態との差分があった場合、その差分をコミット（変更を DOM に適用）します。

つまり、DOM ツリーの構造の修正を行います。

この 3 つの流れについて、React の公式ではレストランを例にして解説されています。

とても分かりやすいのでぜひこちらも読んでみてください。

https://react.dev/learn/render-and-commit#epilogue-browser-paint

そして、React によるこの 3 つの処理が行われた後、ブラウザはその変更を画面に適用します。（ブラウザレンダリング）

コメント: 公式では React のレンダリングとブラウザレンダリングを区別するためにブラウザレンダリングは「ペイント」と呼ばれています

以上の流れにより、画面の表示や更新が行われます。

### Next.js のレンダリングについて

次に、Next.js のレンダリング方式についてまとめていきます。

参考(pages router のページ): https://nextjs.org/docs/pages/building-your-application/rendering

デフォルトでは、Next.js はすべてのページを「プリレンダリング」します。

プリレンダリングとは、Next.js が各ページの HTML をクライアントサイド JavaScript で生成するのではなく、あらかじめ生成しておくことです。
このプリレンダリングにより、パフォーマンスと SEO が向上すると言われています。

生成された HTML は必要 JavaScript コードと関連づけられており、ページがブラウザによってロードされると、その JavaScript コードが実行され、ページが完全にインタラクティブになります。（ハイドレーション）

そして、Next.js には以下の 2 種類のプリレンダリング方式があり、状況に応じて使い分けることができます。

- 静的生成(Static Generation)
  - HTML はビルド時に生成され、リクエストごとに再利用される
- サーバサイドレンダリング(Server-side Rendering)
  - HTML はリクエストごとに生成される

次にこの Static Generation と Server-side Rendering を含めた Next.js のレンダリング種別について解説していきます。

### ハイドレーションとは

ハイドレーションとは、サーバー側からレンダリングされた HTML に紐付けられた JavaScript を実行し、対象のページを完成された状態にすることです。

要はこういう流れです。

1. (SSR などで)サーバー側から HTML が返る
2. クライアント側の JavaScript を実行する(イベントリスナの登録やインタラクティブな動作の追加)

サーバから受け取った初期 HTML は、インタラクティブな機能を持たない乾いた HTML で、そこに、クライアント側で水分（必要な設定や機能）を加えてやるイメージです。

主に SSR のようにサーバー側で HTML を生成して返す場合に使用されます。

### Next.js のレンダリング種別について

React の場合は基本的には CSR での描画を行なっていましたが、Next.js では以下のように様々なレンダリング方式を選択できます。

- SSR
- SSG
- ISR
- CSR

それぞれを簡単に解説していきます。

#### SSR(Server-side Rendering)

Dynamic Rendering とも言われています。

ページの HTML がリクエストごとに生成される方式です。

ページにサーバサイド・レンダリングを使用するには、getServerSideProps という非同期関数をエクスポートします。

この関数は、リクエストごとにサーバーから呼び出されます。

#### SSG(Static Site Generation)

SSG を使用する場合、ページの HTML はビルド時に生成されます。

この HTML はリクエストごとに再利用され、CDN によってキャッシュすることも可能です。

Next.js ではデータが存在しないページだけでなく、getStaticProps や getStaticPaths を使用して build 時にデータを取得・登録して HTML を生成することもできます。

SSG ではリクエストのたびにサーバーがページをレンダリングする必要がないため、レンダリングが非常に高速なのが特徴です。

そのため、基本的にはレンダリング方式として SSG の使用が推奨されています。

#### ISR(Incremental Static Regeneration)

SSG は、事前にページを生成しておき、リクエストごとにその静的なコピーを提供する手法ですが、ISR はこれを更に進化させたものです。

ISR を使うことで、静的ページがあらかじめ生成された後も、一定の時間間隔でそのページを再生成することができます。

これにより、SSG による高速レスポンスを実現しつつも、ある程度のリアルタイム性も提供することができます。

ISR の実装は、Next.js のページごとに revalidate というパラメータを設定することで行います。
このパラメータには、再生成の間隔を指定します。例えば、revalidate: 60 と設定すると、60 秒ごとにページが再生成されます。

つまり、ISR は、静的なコンテンツを効果的に提供しつつ、定期的に最新の情報を反映させる仕組みを提供する機能です。

#### CSR

CSR は（先ほど説明した）React のデフォルトのレンダリング方式です。

ブラウザ上で JavaScript を実行して DOM を生成しコンテンツを表示させます。

Next.js では useEffect フックを使用するなどして CSR を実装することができます。

参考 1: https://nextjs.org/docs/app/building-your-application/rendering
参考 2: https://nextjs.org/docs/pages/building-your-application/rendering

※本の内容も参考に

### RSC とは何か？

- https://twitter.com/newbee1939/status/1680942624600117250

連休最終日、React Server Components および Next.js の App Router(App Directory)について、主に以下の記事を使って学んだので、自分なりに理解したことを連投でまとめてみます。

- https://zenn.dev/suzu_4/articles/2e6dbb25c12ee5
- https://zenn.dev/uhyo/articles/react-server-components-multi-stage
- https://zenn.dev/g4rds/articles/287c53498d17a1
- https://eh-career.com/engineerhub/entry/2023/07/14/093000

そもそも、Next.js には SSR（Server Side Rendering）という、React を Node.js 上で動作させる仕組みがあります。SSR を使用することで、初期レンダリング時にサーバー側で HTML を生成してレンダリングすることができます。これにより、コンテンツがしっかりと詰まった状態の初期画面が表示されるため、SEO の面でも効果があるとされています。

しかし、SSR には「ハイドレーション」という仕組みが存在します。これは、SSR で HTML 等をレンダリングした後に、フロントエンドでイベントリスナを登録したり、インタラクティブな動作を追加する機能です。

これは SSR を実現する上でとても重要な機能です。

しかし、ハイドレーションを実行するためには、フロント側にコンポーネント（アプリ）全体のデータを送信する必要があり、その結果 bundle サイズが増加してしまいます。bundle サイズの増加はパフォーマンスの悪化に直結します。

そこで生まれたのが React Server Components とその仕組みを組み込んだ Next.js の App Router（App Directory）です。

React Server Components は React のコンポーネントをサーバー側で実行する仕組みです。React アプリケーションをサーバー側で処理する部分とクライアント側で処理する部分に分け、サーバー側で実行可能なコンポーネントをサーバー側で実行することで、クライアント側のコードが減る（クライアントに送る JavaScript の bundle サイズが減る）ため、パフォーマンスの改善が期待できます。

そして、Next.js の App Router は React Server Components を組み込んだ仕組みです。

App Router では、デフォルトで RSC（React Server Components）が適用されます。つまり、作成したコンポーネントがサーバー側で実行されるということです。クライアント側で実行させるには、`use client` を定義する必要があります。

できるだけサーバー側に処理を寄せることで、パフォーマンスの改善を図るという意図が読み取れます。

React Server Components と SSR は同じ仕組みでも相反する仕組みでもなく、それぞれ異なる責任領域を持つ仕組みです。

これらを組み合わせることで、SSR によって初期表示を速くしつつ、React Server Components によって（今まではクライアント単体だった）コンポーネントをクライアントとサーバー用に分割し、できるだけサーバー側で処理させることで、SSR のハイドレーション時の bundle サイズを最小限に抑えることができます。

React Server Components と SSR を組み合わせることで、SEO の最適化を図る一方で、パフォーマンスの面でも最適化を図ることが可能です。

- https://nextjs.org/docs/app
- https://nextjs.org/docs/app/building-your-application/upgrading/app-router-migration

### App Router とは何か？

https://nextjs.org/docs/app/building-your-application/rendering

### SSR と App Router(RSC)の違いについて

SSR（サーバーサイドでのレンダリング＋ハイドレーション）

RSC （サーバーサイドでのレンダリングのみ？）

SSG は？サーバーサイドでのレンダリングのみではなく？

### SSR ト App Router(RSC)をどう使い分ける？

### App Router(RSC)のサンプル実装

### React の Suspense とは何か？

https://eh-career.com/engineerhub/entry/2023/07/14/093000

### まとめ

### 参考資料
