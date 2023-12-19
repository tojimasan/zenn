---
title: "不要なuseStateを削除する"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react", "javascript"]
published: false
---

# はじめに

React を書き始めてからよく使うカスタムフックの中で `useState` と `useEffect` があります。

日々開発する中で、やってしまいがちな fetch したデータを `useState` で保存する方法。この中に含まれた不要な `state` の削除を目指します。

:::message
state や useState, useEffect について詳しく知りたい方は、公式ドキュメントをオススメします

https://ja.react.dev/learn/state-a-components-memory
https://ja.react.dev/reference/react/useState
https://ja.react.dev/reference/react/useEffect
:::

## 対象読者

- React の useState や useEffect などの基本的なカスタムフックを理解している
- 初心者〜歴１年程度のフロントエンドエンジニア

# fetch したデータを state に持たせる

@[stackblitz](https://stackblitz.com/edit/stackblitz-starters-4l9wer?embed=1&file=src%2FApp.tsx)

`useEffect` で特定のエンドポイントからデータを取得し、必要なデータを state に入れる処理になります。
エンドポイントは呼び出されてから１秒後にダミーデータを返す実装となっております。

## コードが抱える課題

- 3 回もレンダリングされる
- set 関数の変更が反映されない

### 3 回もレンダリングされる

この App コンポーネントは

1. 初回レンダリング
2. 1 つ目の `useEffect` 内で実行される `setData` をトリガーとする再レンダリング
3. 2 つ目の `useEffect` 内で実行される `setUsers` をトリガーとする再レンダリング

3 回レンダリングが実行されます。1 つ目の `useEffect` はデータを取得するため外部との接続に必要です。一方で 2 つ目の `useEffect` や関連する `useState` は無くせるように感じられます。

また、App コンポーネントに子コンポーネントがあれば、その分レンダリングコストが増える実装です。

### set 関数の変更が反映されない

先ほどのコードでは `setUsers` が公開されており、開発者はどこでも `users` の値を書き換えられます。以下のようなコードが JSX 内にあったとします。

```jsx
...
<Button onClick={() => setUsers(usersLivedInUSA)}>
  live in USA
</Button>
```

この機能の開発者は「アメリカに住んでいるユーザのみを表示する」という目的で用意したのでしょう。
しかし、ボタンをクリックしても初期のユーザが表示されるばかりです。
なぜでしょうか。順を追って見てみましょう。

1. クリックによって再レンダリングがトリガー
2. `users` として `newUsers` がセットされた state が React からコンポーネントに渡される
3. useEffect の `fetch` が再実行(fetch に react-query を使用)
4. `fetch` の結果が `users` にセットされる

アメリカ在住の `users` を表示したいのに、いつまでも最初の `users` が表示されます

# 改善

@[stackblitz](https://stackblitz.com/edit/stackblitz-starters-nettjd?file=src%2FApp.tsx)

2 つ目の `useEffect` を削除し、`users` は `data` の有無で持たせるデータを買えるようにしました。

```tsx
const users: User[] = data ? data : [];
```

この修正により、レンダリング回数は１回減り、必要のない state を減らすことができます。

# まとめ

この記事では、React でよく使われる `useState` フックについて、不要な state を削除する方法について説明しました。

fetch したデータを state に保存する際に、不要な state を削除し、また不要なレンダリングを減らすため、余分な `useEffect` フックを削除しました。

これにより、コンポーネントのパフォーマンスを向上させることができます。

# 参考

https://tkdodo.eu/blog/dont-over-use-state
