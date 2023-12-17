---
title: "不要なuseStateを削除する"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react", "javascript"]
published: false
---

# はじめに

## state が増えることの課題

コンポーネントに state が増えることによって、コンポーネントの複雑性が上がったり、バグが発生する可能性が高まる

(state や useState については公式ドキュメントを見るように誘導する)

[state：コンポーネントのメモリ](https://ja.react.dev/learn/state-a-components-memory)
[useState](https://ja.react.dev/reference/react/useState)

## 対象読者

- React の useState や useEffect などの基本的なカスタムフックを理解している
- 歴１年程度のフロントエンドエンジニア

# 例: fetch したデータを state に持たせる

@[stackblitz](https://stackblitz.com/edit/stackblitz-starters-4l9wer?embed=1&file=src%2FApp.tsx)
(簡単にコードを解説)

## コードが抱える課題
- 3回もレンダリングされる
  - 子コンポーネントがあれば、レンダリングコストがさらに増える
- set関数の変更が反映されない
先ほどのコードではsetUsersが公開されており、開発者はどこでもusersの値を書き換えられます。以下のようなコードがJSX内にあったとします。
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
2. users として newUsers がセットされたstateがReactからコンポーネントに渡される
3. useEffect の fetchが再実行
4. fetchの結果がusersにセットされる

アメリカ在住のusersを表示したいのに、いつまでも最初のusersが表示されます

# 改善

@[stackblitz](https://stackblitz.com/edit/stackblitz-starters-nettjd?file=src%2FApp.tsx)

- 不要な useState と useEffect を削除
  - users は data を取得できていれば、その値を使う

# まとめ
