# React Server Components と App Router を今度こそちゃんと理解する

「仕事で React や Next.js を使っているのに RSC とか App Router とか Suspense とか何にも分からん。。やばい。。。」と焦りを覚えたので、勉強しつつ Qiita にまとめてみました。

この記事を読めば、以下の項目が理解できるようになるでしょう。

- React や Next.js の基礎知識
- React Server Components とは何か？
- React Server Components のレンダリングの流れ
- Suspense とは何か？
- App Router とは何か？

少しでも私と同じような思いを抱えている方々の助けになれば幸いです。

## React と Next.js について

いきなり React Server Components から の深いところから話しても難しい思うので、順を追って説明していきます。

まずは React と Next.js について簡単に解説します。

React とは、UI を簡単に構築するための JavaScript ライブラリです。

「コンポーネント」という概念を使って宣言的に UI を定義することで、簡単に画面の構築を行うことができます。

そして、Next.js とは React のフレームワークです。

React の機能をさらに拡張してより使いやすくしたものと捉えて良いでしょう。

## React のレンダリングについて

React は、create-react-app で作成された初期状態では、CSR（クライアントサイドレンダリング）でレンダリングを行います。

CSR とは、ブラウザ上で JavaScript を実行して DOM を生成しコンテンツを表示させる方法です。

ページの初期ロード時にはコンテンツは何も表示されず（白い画面）、ブラウザでの JavaScript の実行後に画面が表示されます。

より詳細に説明すると、React でのレンダリングは、以下のステップで行われます。

[画像 1]

### 1. レンダリングのトリガーを検知

レンダリングのきっかけとなるトリガーを検知します。

ここでのトリガーとは、以下の二つです。

- コンポーネントの初期レンダリング（画面の初期ロード）
- コンポーネントの状態（state）の更新

このいずれかのトリガーを検知したとき、React はレンダリングを開始します。

### 2. ブラウザレンダリングする内容の決定

次に、実際にレンダリングを行います。

React における「レンダリング」は主に以下の３つを行います。

1. 対象のコンポーネントの呼び出し
2. 以前のコンポーネントの状態との比較
3. コミット（ブラウザレンダリング）する内容の決定

1.の「対象のコンポーネント」について、最初のレンダリングではルートコンポーネントを呼び出し、それ以降のレンダリングでは状態更新がレンダリングのトリガーとなった関数コンポーネントを呼び出します。

そして 2 で以前のコンポーネントの状態との差分を計算し、（そもそも画面を更新する必要があるか否かも含めて）画面更新の内容を決定します。

そして、差分が検知されて画面を更新する必要がある場合、次の手順 3.を実行します。

つまり、React におけるレンダリングとは、「レンダリング対象のコンポーネントを呼び出して前回の内容との差分を比較し、何をブラウザレンダリング（commit）をするかを決定すること」と言えるでしょう。

同じ「レンダリング」という名前が付いているので混同しがちですが、「ブラウザレンダリング（画面への描画）」とは違う概念なのでしっかりと区別するようにしましょう。

順番としては、「React のレンダリング」をした後に「ブラウザのレンダリング」を行なっているイメージです。

### 3. 変更を DOM に適用

2.で以前の状態との差分があった場合、その差分をコミット（変更を DOM に適用）します。

つまり、DOM ツリーの構造の修正を行います。

この 3 つの流れについて、React の公式ではレストランを例にして解説されています。
とても分かりやすいのでぜひ読んでみてください。
https://react.dev/learn/render-and-commit#epilogue-browser-paint

そして、React によるこの 3 つの処理が行われた後、ブラウザはその変更を画面に適用します。（ブラウザレンダリング）

コメント: React の公式では React のレンダリングとブラウザレンダリングを区別するためにブラウザレンダリングは「ペイント」と呼ばれています

以上の流れにより、React で画面の表示や更新が行われます。

<ここまでのまとめ>

- React は UI を簡単に構築するための JavaScript ライブラリ
- Next.js は React のフレームワーク
- React は以下の流れでレンダリング（CSR）を行う
  - 1. レンダリングのトリガーを検知
  - 2. ブラウザレンダリングする内容の決定
  - 3. 変更を DOM に適用

## Next.js のレンダリングについて

次に、Next.js のレンダリング方式についてまとめていきます。

2023 年 9 月現在、Next.js には以下の二つのモード（ルーティング方式）があります。
もともとは Pages Router しかありませんでしたが、最近 App Router が追加されました。

- Pages Router
- App Router

最初に Pages Router でのレンダリングについてまとめていきます。

### Next.js のレンダリングについて(Pages Router)

デフォルトでは、Next.js はすべてのページを「プリレンダリング」します。

プリレンダリングとは、Next.js が各ページの HTML をクライアントサイド JavaScript で生成するのではなく、あらかじめ生成しておくことです。

このプリレンダリングにより、パフォーマンスと SEO が向上すると言われています。

生成された HTML は必要な JavaScript と関連づけられており、ページがブラウザによってロードされると、その JavaScript が実行され、ページが完全にインタラクティブになります。（これをハイドレーションと言います）

Next.js（の Pages Router）には以下の 2 種類のプリレンダリング方式があり、状況に応じて使い分けることができます。

- 静的生成(Static Generation)
  - HTML はビルド時に生成され、リクエストごとに再利用される
- サーバサイドレンダリング(Server-side Rendering)
  - HTML はリクエストごとに生成される

### ハイドレーションとは何か？

ハイドレーションとは、サーバー側からレンダリングされた HTML に紐付けられた JavaScript を実行し、対象のページを完成された状態にすることです。

要はこういう流れです。

1. (SSR などで)サーバー側から HTML が返る
2. クライアントに送信された JavaScript を実行する(イベントリスナの登録やインタラクティブな動作の追加)

サーバから受け取った初期 HTML は、インタラクティブな機能を持たない乾いた HTML で、そこに、クライアント側で水分（必要な設定や機能）を加えてやるイメージです。

主に SSR のようにサーバー側で HTML を生成して返す場合に使用されます。

次に Static Generation と Server-side Rendering を含めた Next.js（Pages Router）のレンダリング方式について解説していきます。

### Next.js のレンダリング種別について(Pages Router)

React の場合は基本的には CSR での描画を行なっていましたが、Next.js（の Pages Router）では以下のように様々なレンダリング方式を選択できます。

- SSR
- SSG
- ISR
- CSR

それぞれを簡単に解説していきます。

#### SSR(Server-side Rendering)

SSR は Dynamic Rendering とも言われています。

ページの HTML がリクエストごとに生成される方式です。

サーバー側で生成された生の HTML が JavaScript の実行前にブラウザで表示されるため、画面の初期表示速度を速めることができます。

これはユーザー体験の向上はもちろんのこと、SEO にも効果があると言われています。

サーバサイド・レンダリングを使用するには、getServerSideProps という非同期関数をエクスポートします。

#### SSG(Static Site Generation)

SSG を使用する場合、ページの HTML はビルド時に生成されます。

この HTML はリクエストごとに再利用され、CDN によってキャッシュすることも可能です。

Next.js では、データが存在しないページ（静的 HTML）だけでなく、getStaticProps や getStaticPaths を使用して build 時にデータを取得・登録して HTML を生成することもできます。

SSG ではリクエストのたびにサーバーがページをレンダリングする必要がないため、レンダリングが非常に高速なのが特徴です。

そのため、基本的にはレンダリング方式として SSG の使用が推奨されています。

#### ISR(Incremental Static Regeneration)

SSG は、事前にページを生成しておき、リクエストごとにその静的なコピーを提供する手法ですが、ISR はこれを更に発展させたものです。

ISR を使うことで、静的ページがあらかじめ生成された後も、一定の時間間隔でそのページを再生成することができます。

これにより、SSG による高速レスポンスを実現しつつも、ある程度のリアルタイム性も提供することができます。

ISR の実装は、Next.js のページごとに revalidate というパラメータを設定することで行います。
このパラメータには、再生成の間隔を指定します。例えば、revalidate: 60 と設定すると、60 秒ごとにページが再生成されます。

つまり、ISR は、静的なコンテンツを効果的に提供しつつ、定期的に最新の情報を反映させる仕組みを提供する機能です。

comment: SSR と SSG の間の技術と捉えてもいいかもしれません

#### CSR

CSR は（先ほど説明した）React のデフォルトのレンダリング方式です。

ブラウザ上で JavaScript を実行して DOM を生成しコンテンツを表示させます。

Next.js では useEffect フックを使用するなどして CSR を実装することができます。

---

次に、いよいよ Pages Router に続く Next.js 二つ目のモードである、App Router について解説していきます。

ただ、App Router を理解するためには、そのベースの技術である RSC（React Server Components）を理解する必要があります。

そのため、まずは「RSC とは何か」について説明していきます。

<ここまでのまとめ>

- React は UI を簡単に構築するための JavaScript ライブラリ
- Next.js は React のフレームワーク
- React は以下の流れでレンダリング（CSR）を行う
  - 1. レンダリングのトリガーを検知
  - 2. ブラウザレンダリングする内容の決定
  - 3. 変更を DOM に適用
- Next.js には以下の二つのモード（ルーティング方式）がある
  - Pages Router
  - App Router
- Next.js の Pages Router では以下の 4 つのレンダリング方式を選択できる
  - SSR
  - SSG
  - ISR
  - CSR

### RSC（React Server Components）とは何か？

もともと React には、先ほど説明した CSR しかありませんでした。

しかし、CSR の場合、クライアントに全てのコンポーネントのリソース（JavaScript）を送信する必要があり、クライアントのパフォーマンス悪化が懸念されていました。

そこで誕生したのが、RSC（React Server Components）です。

RSC とは、一言で言うと、コンポーネントを「サーバー側でレンダリングされるコンポーネント」と「クライアント側でレンダリングされるコンポーネント」に分ける技術です。

これまで React には「クライアントコンポーネント」しかありませんでした。しかし RSC では、どのコンポーネントをサーバー専用にし、どのコンポーネントをクライアント専用にするかを選択できます。

[画像]

RSC により、データ取得等をより DB に近いサーバー側で実行できる＋サーバーコンポーネントや依存パッケージの分クライアント側に送信する JavaScript のサイズ（bundle サイズ）を減らせるため、パフォーマンスが向上すると言われています。

さらに、RSC には以下のような特徴があります。

- サーバー側からより高速にデータ取得が可能になる（クライアントからのリクエスト量も減る）
- console.log はブラウザのコンソールではなく、サーバーのコンソールに情報を記録する
- onClick や onChange などのイベントリスナーは使用できない
- 状態管理（useState）と効果管理（useEffect）は使用できない
- サーバーコンポーネントはクライアントコンポーネントをインポートしてレンダリングできるが、クライアントコンポーネントはその中のサーバーコンポーネントをレンダリングできない

comment: ただし、RSC を使えば無条件にサイズが減少するというわけではないようです。詳しくは以下を参考にしてください。https://qiita.com/uhyo/items/06b0cd7292256f66d7b7

### App Router とは何か？

App Router とは Next.js13 で追加された、新しいルーターの実装です。（Next.js の新しいバージョン）

App Router では、デフォルトで RSC（React Server Components）が適用されます。

つまり、作成したコンポーネントがサーバー側で実行されるということです。

クライアント側で実行させるには、コンポーネントのトップに`use client` を定義する必要があります。

comment: できるだけサーバー側に処理を寄せることで、パフォーマンスの改善を図るという意図が読み取れます。基本的にはサーバーコンポーネントとして実装して、必要な箇所だけクライアントで実行させると良いでしょう

現在では、Pages Router より App Router の利用が推奨されています。

<ここまでのまとめ>

- React は UI を簡単に構築するための JavaScript ライブラリ
- Next.js は React のフレームワーク
- React は以下の流れでレンダリング（CSR）を行う
  - 1. レンダリングのトリガーを検知
  - 2. ブラウザレンダリングする内容の決定
  - 3. 変更を DOM に適用
- Next.js には以下の二つのモード（ルーティング方式）がある
  - Pages Router
  - App Router
- Next.js の Pages Router では以下の 4 つのレンダリング方式を選択できる
  - SSR
  - SSG
  - ISR
  - CSR
- React Server Components とは、コンポーネントを「サーバー側でレンダリングされるコンポーネント」と「クライアント側でレンダリングされるコンポーネント」に分ける技術
- App Router では、デフォルトで作成したコンポーネントがサーバーコンポーネントになる

### SSR と App Router(RSC)の違いについて

次に、SSR と RSC の違いについて、図を交えつつ解説します。

SSR は、以下のようにレンダリングされます。

[図を貼る]

1. サーバー側で全体をレンダリングし、生の HTML にする
2. 生成した HTML を DOM に反映させてクライアント側で表示（ペインティング）する（初期表示を早める）
3. bundle した JavaScript（コンポーネント）をクライアントに送信しハイドレーションを行う

最初にブラウザ側で HTML を生成してクライアント側に反映させることで初期表示を早めるのが最大の特徴です。

RSC では、以下のようにレンダリングされます。

[画像]

1. サーバー側でサーバーコンポーネントをレンダリングする
2. サーバーコンポーネントの HTML とクライアントコンポーネントの JavaScript をクライアントに送信する
3. クライアントコンポーネントをレンダリングする
4. 生成した HTML を DOM に反映させてクライアント側で表示する

大きな違いは以下の二つです。
・SSR の場合は初期表示が速い
・RSC の場合はサーバーとクライアントでそれぞれのコンポーネントがレンダリングされる
・SSR の方がクライアントに送信される JavaScript の量が多い

この SSR と RSC は交わらない技術ではなく、組み合わせて使用することもできます。
SSR と RSC を組み合わせた場合、処理の流れは以下のようになります。

[画像貼る]

1. サーバー側でサーバーコンポーネントをレンダリングする
2. サーバー側でクライアントコンポーネントもレンダリングする(ここが SSR 要素。本来クライアント側で行うレンダリングをサーバー側でも行う)(これにより従来と同じただの HTML 文字列をサーバー側で得られる)
3. 生成したサーバーコンポーネントとクライアントコンポーネントの HTML をクライアント側に送信して DOM に反映させてクライアント側で表示させる
4. クライアント側に bunde した JavaScript（クライアントコンポーネント）を送信し、レンダリングとハイドレーションを実行する

このように、SSR と RSC を組み合わせることで、初期表示を早めつつ、クライアント側に送信する JavaScript の量を抑えることができます。

そのため、基本的には RSC 単体で使うよりも、RSC と SSR を組み合わせて使うことの方が多いでしょう。

### React Server Components とデータの取得について

ここまでは、特に「外部からのデータの取得」は意識せずに解説してきました。

しかし、実際のアプリケーションだと外部（DB や API）からデータを取得して使用することが多いでしょう。

次に、その場合にどういった流れになるのかについて解説します。

もともと、クライアント側（SSR しないアプリケーション）では useState 等を使ってローディング中の状態を表現することで、コンポーネント単位での非同期的なデータ取得が可能でした。

しかし、SSR をする場合（初期ペインティングの時点で表示用のデータが欲しい場合）、useState 等を使ってサーバー側でコンポーネント単位で非同期的なデータ取得ができない問題がありました。

Next.js の getServerSideProps を使ってデータの取得はできていましたが、データ取得が「同期的」という課題がありました。
これだと、ページをユーザーに表示する前に、サーバーでのデータ取得をすべて完了させる必要があるため、ユーザーに対する画面の表示が遅れてしまうことがあります。

しかし、この状況が Suspense と React Server Components の登場で変わります。

Suspense とは、useState 等に頼らずに「ローディング中」を宣言的に表現できる機能です。

例えば、以下のようなコンポーネントを用意します。

これを画面に表示すると、以下のようになります。

[GIF 貼る]

useState を使わずに「ローディング中」を表現できていることがわかるでしょう。

また、React Server Components では、async/await（非同期関数）を使用することができます。

このように Suspense と React Server Components を使うことで、SSR を使用するサーバー側でも（getSSP を使わない）コンポーネント単位の非同期的なデータ取得ができるようになりました。

これにより、完全にデータ取得をする前にユーザーは画面を見ることができます。
従来の SSR よりも表示速度を早めることができるでしょう。

Next.js の場合、Streaming SSR という機能でこれが使われています。

[画像貼る]

ただし、Streaming SSR だけだと、ハイドレーションにより、サーバー側およびクライアントの両方でデータ取得が走ってしまう問題がありました。

この問題を解消するのが RSC です。

RSC により、コンポーネントをクライアントコンポーネントとサーバーコンポーネントに分けることができるため、データ取得をサーバ側のみに制限することができます。

これにより、データ取得がサーバー側とクライアント側の両方で走ることはありません。

[画像貼る]

SSR でデータ取得をする場合、RSC を使用してサーバーコンポーネント内で Suspense を使って loading 中を表現しつつデータ取得をするのが現状の最適解と言えるでしょう。

<ここまでのまとめ>

- React は UI を簡単に構築するための JavaScript ライブラリ
- Next.js は React のフレームワーク
- React は以下の流れでレンダリング（CSR）を行う
  1. レンダリングのトリガーを検知
  2. ブラウザレンダリングする内容の決定
  3. 変更を DOM に適用
- Next.js には以下の二つのモード（ルーティング方式）がある
  - Pages Router
  - App Router
- Next.js の Pages Router では以下の 4 つのレンダリング方式を選択できる
  - SSR
  - SSG
  - ISR
  - CSR
- React Server Components とは、コンポーネントを「サーバー側でレンダリングされるコンポーネント」と「クライアント側でレンダリングされるコンポーネント」に分ける技術
- Next.js の App Router では、デフォルトで作成したコンポーネントがサーバーコンポーネントになる
  - クライアントコンポーネントにするには`use client;`を記述する必要がある
- RSC と SSR を組み合わせることで、初期表示を早めつつ、クライアント側に送信する JavaScript の量を抑えることができる
- Suspense は、useState 等に頼らずに「ローディング中」を表現できる機能です。
- Suspense と React Server Components を使うことで、SSR を使用するサーバー側でもコンポーネント単位の非同期的なデータ取得が可能となる

### 実際に RSC・App Router を触ってみる

次に、実際にアプリを動かしつつ、挙動を確かめていこうと思います。

簡単な操作なので、ぜひ読者の方も実際に試してみてください。

前提として、Node.js,npm,TypeScript は使える状態になっているとします。

https://www.engilaboo.com/build-environment-of-typescript/

まずは、以下のコマンドで Next.js をインストールします。

```
npx create-next-app@latest .
```

最初に、シンプルなサーバーコンポーネントとクライアントコンポーネントを作っていきます。

以下のファイルをそれぞれ作成してください。

[app/page.tsx]

```tsx
import { ClientComponent } from "./components/ClientComponent";
import { ServerComponent } from "./components/ServerComponent";

export default async function Home() {
  return (
    <div
      style={{
        display: "flex",
        justifyContent: "center",
        alignItems: "center",
        marginTop: "200px",
      }}
    >
      <ClientComponent />
      <ServerComponent />
    </div>
  );
}
```

[app/components/ServerComponent.tsx]

```tsx
export async function ServerComponent() {
  const boxStyle = {
    width: "400px",
    height: "300px",
    backgroundColor: "#006400",
    display: "flex",
    justifyContent: "center",
    alignItems: "center",
  };

  const textStyle = { color: "white", footSize: "larger", fontWeight: "bold" };

  console.log("Server Componentを実行しています");

  return (
    <div style={boxStyle}>
      <p style={textStyle}>Server Component</p>
    </div>
  );
}
```

[app/components/ClientComponent.tsx]

```tsx
"use client";

export function ClientComponent() {
  const boxStyle = {
    width: "400px",
    height: "300px",
    backgroundColor: "#ffff00",
    display: "flex",
    justifyContent: "center",
    alignItems: "center",
  };

  const textStyle = { footSize: "larger", fontWeight: "bold", color: "black" };

  console.log("Client Componentを実行しています");

  return (
    <div style={boxStyle}>
      <p style={textStyle}>Client Component</p>
    </div>
  );
}
```

App Router ではコンポーネントはデフォルトでサーバーコンポーネントになります。
クライアントコンポーネントにするには、ページの最初に`use client;`を記述する必要があります。

ここでは、サーバーコンポーネントとクライアントコンポーネントをそれぞれ作成し、root コンポーネントで両方を読み込みます。

そして`npm run dev`でアプリケーションを実行します。
localhost:3000 にアクセスすると、以下の画面が表示されるでしょう。

[画像貼る]

まずは開発者ツールのコンソールを開きます。
すると、クライアントコンポーネントだけが表示されていることが分かると思います。

逆にターミナルを見ると、サーバー側でサーバーコンポーネントが実行されていることが分かると思います。（SSR されているので、クライアントコンポーネントも実行されていますが）

さらに、Network タブを見てみると、page.js は 39.2kB になっています。

次に、以下のように全てのコンポーネントに`use client;`を付けます。

[app/page.tsx]

```tsx
"use client";

import { ClientComponent } from "./components/ClientComponent";
import { ServerComponent } from "./components/ServerComponent";

export default async function Home() {
  return (
    <div
      style={{
        display: "flex",
        justifyContent: "center",
        alignItems: "center",
        marginTop: "200px",
      }}
    >
      <ClientComponent />
      <ServerComponent />
    </div>
  );
}
```

[app/components/ServerComponent.tsx]

```tsx
"use client";

export async function ServerComponent() {
  const boxStyle = {
    width: "400px",
    height: "300px",
    backgroundColor: "#006400",
    display: "flex",
    justifyContent: "center",
    alignItems: "center",
  };

  const textStyle = { color: "white", footSize: "larger", fontWeight: "bold" };

  console.log("Server Componentを実行しています");

  return (
    <div style={boxStyle}>
      <p style={textStyle}>Server Component</p>
    </div>
  );
}
```

[app/components/ClientComponent.tsx]

```tsx
"use client";

export function ClientComponent() {
  const boxStyle = {
    width: "400px",
    height: "300px",
    backgroundColor: "#ffff00",
    display: "flex",
    justifyContent: "center",
    alignItems: "center",
  };

  const textStyle = { footSize: "larger", fontWeight: "bold", color: "black" };

  console.log("Client Componentを実行しています");

  return (
    <div style={boxStyle}>
      <p style={textStyle}>Client Component</p>
    </div>
  );
}
```

この状態で再度画面を表示します。そして、ネットワークタブを見ると、page.js のサイズが 40.6kB になっています。
ServerComponent をクライアントコンポーネンにした分、サイズが増加していることが分かると思います。

今度は逆に、以下のように全てのコンポーネントから`use client`を消します。

[app/page.tsx]

```tsx
import { ClientComponent } from "./components/ClientComponent";
import { ServerComponent } from "./components/ServerComponent";

export default async function Home() {
  return (
    <div
      style={{
        display: "flex",
        justifyContent: "center",
        alignItems: "center",
        marginTop: "200px",
      }}
    >
      <ClientComponent />
      <ServerComponent />
    </div>
  );
}
```

[app/components/ServerComponent.tsx]

```tsx
export async function ServerComponent() {
  const boxStyle = {
    width: "400px",
    height: "300px",
    backgroundColor: "#006400",
    display: "flex",
    justifyContent: "center",
    alignItems: "center",
  };

  const textStyle = { color: "white", footSize: "larger", fontWeight: "bold" };

  console.log("Server Componentを実行しています");

  return (
    <div style={boxStyle}>
      <p style={textStyle}>Server Component</p>
    </div>
  );
}
```

[app/components/ClientComponent.tsx]

```tsx
export function ClientComponent() {
  const boxStyle = {
    width: "400px",
    height: "300px",
    backgroundColor: "#ffff00",
    display: "flex",
    justifyContent: "center",
    alignItems: "center",
  };

  const textStyle = { footSize: "larger", fontWeight: "bold", color: "black" };

  console.log("Client Componentを実行しています");

  return (
    <div style={boxStyle}>
      <p style={textStyle}>Client Component</p>
    </div>
  );
}
```

`use client;`を消して、サーバーコンポーネントに変更しました。

この状態で再度画面を表示します。そして、ネットワークタブを見ると、page.js が消えていることが分かると思います。
これは、コンポーネントの全てのレンダリングがサーバーサイドで行われたことを示しています。

次に、Suspense を使ってサーバーコンポーネントでのデータ取得をシミュレーションします。

それぞれ以下のように変更してください。

[app/page.tsx]

```tsx
import { Suspense } from "react";
import { ClientComponent } from "./components/ClientComponent";
import Loading from "./components/loading";
import { ServerComponent } from "./components/ServerComponent";

export default async function Home() {
  return (
    <div
      style={{
        display: "flex",
        justifyContent: "center",
        alignItems: "center",
        marginTop: "200px",
      }}
    >
      <ClientComponent />
      <Suspense fallback={<Loading />}>
        <ServerComponent />
      </Suspense>
    </div>
  );
}
```

[app/components/ServerComponent.tsx]

```tsx
const sleep = async (ms: number) => {
  return new Promise((res) => setTimeout(res, ms));
};

export async function ServerComponent() {
  console.log("ServerComponentを実行しています(sleepの前)");

  // データの取得をイメージ
  await sleep(3000);

  const boxStyle = {
    width: "400px",
    height: "300px",
    backgroundColor: "#006400",
    display: "flex",
    justifyContent: "center",
    alignItems: "center",
  };

  const textStyle = { color: "white", footSize: "larger", fontWeight: "bold" };

  console.log("Server Componentを実行しています(sleepの後)");

  return (
    <div style={boxStyle}>
      <p style={textStyle}>Server Component</p>
    </div>
  );
}
```

[app/components/ClientComponent.tsx]

```tsx
"use client";

export function ClientComponent() {
  const boxStyle = {
    width: "400px",
    height: "300px",
    backgroundColor: "#ffff00",
    display: "flex",
    justifyContent: "center",
    alignItems: "center",
  };

  const textStyle = { footSize: "larger", fontWeight: "bold", color: "black" };

  console.log("Client Componentを実行しています");

  return (
    <div style={boxStyle}>
      <p style={textStyle}>Client Component</p>
    </div>
  );
}
```

さらに、以下の loading.tsx も追加します。

```tsx
export default function Loading() {
  const boxStyle = {
    width: "400px",
    height: "300px",
    backgroundColor: "#CACACA",
    display: "flex",
    justifyContent: "center",
    alignItems: "center",
  };

  const textStyle = { color: "black", fontSize: "larger", fontWeight: "bold" };

  return (
    <div style={boxStyle}>
      <p style={textStyle}>...Loading</p>
    </div>
  );
}
```

この状態で画面を表示すると、以下のようになります。

[GIF 貼る]

サーバー側で非同期的にデータを取得した上で画面の表示ができていることが分かると思います。

## まとめ

最後にもう一度内容をまとめておきます。

- React は UI を簡単に構築するための JavaScript ライブラリ
- Next.js は React のフレームワーク
- React は以下の流れでレンダリング（CSR）を行う
  1. レンダリングのトリガーを検知
  2. ブラウザレンダリングする内容の決定
  3. 変更を DOM に適用
- Next.js には以下の二つのモード（ルーティング方式）がある
  - Pages Router
  - App Router
- Next.js の Pages Router では以下の 4 つのレンダリング方式を選択できる
  - SSR
  - SSG
  - ISR
  - CSR
- React Server Components とは、コンポーネントを「サーバー側でレンダリングされるコンポーネント」と「クライアント側でレンダリングされるコンポーネント」に分ける技術
- Next.js の App Router では、デフォルトで作成したコンポーネントがサーバーコンポーネントになる
  - クライアントコンポーネントにするには`use client;`を記述する必要がある
- RSC と SSR を組み合わせることで、初期表示を早めつつ、クライアント側に送信する JavaScript の量を抑えることができる
- Suspense は、useState 等に頼らずに「ローディング中」を表現できる機能です。
- Suspense と React Server Components を使うことで、SSR を使用するサーバー側でもコンポーネント単位の非同期的なデータ取得が可能となる

## おわりに

SSR に回帰する？
https://www.publickey1.jp/blog/23/webssrdenoisomorphic_javascriptuniversal_javascript.html

https://www.publickey1.jp/blog/23/astro_30javascriptspa.html

### 主な参考資料

参考: 一言で理解する React(https://zenn.dev/uhyo/articles/react-server-components-multi-stage)
参考: Next.js 公式ドキュメント(https://nextjs.org/docs)
参考: What's "Next" JS Meetup(https://www.youtube.com/watch?v=WHMm6w41_WI&ab_channel=TimeeEngineering)
参考: React Server Components の仕組み：詳細ガイド(https://postd.cc/how-react-server-components-work/)
参考: Nextjs で理解する React Server Components 徹底解説【React18】(https://youtu.be/A78v05JSyqg?si=EJiKhE35K11TbcGe)
参考: Understanding React Server Components(https://vercel.com/blog/understanding-react-server-components)
参考: Next.js から学ぶ Web レンダリング ~React 誕生以前から App Router with RSC までの流れ~(https://zenn.dev/suzu_4/articles/2e6dbb25c12ee5)
参考: Render and Commit(https://react.dev/learn/render-and-commit)
参考: Making Sense of React Server Components(https://www.joshwcomeau.com/react/server-components/)

※ChatGPT で誤字脱字等の修正
※読者の視点を考える
