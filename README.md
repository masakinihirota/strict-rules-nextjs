
Antigravityを使う前に: Next.js用に人間には厳しいルールを適用したサンプル #Gemini - Qiita

https://qiita.com/masakinihirota/items/dbbd8f897114a422d83a

保存

---

## 📌 Barrel Export（バレルエクスポート）とは？
Barrel Export は、ディレクトリの入口（通常 `index.ts`）にそのディレクトリ配下の複数ファイルからの再エクスポート（re-export）をまとめたファイルを置く手法です。
パスを短く・分かりやすくすることで、利用側のインポートをシンプルにする目的で使われます。

例:
- 個別インポート（深いパス）:
  ```ts
  import { ProfileList } from "@/components/profile-list/profile-list";
  ```
- バレル経由（短くする）:
  ```ts
  import { ProfileList } from "@/components/profile-list";
  ```

---

## ✅ 利点
- インポートが短く、見通しが良くなる（可読性向上）
- モジュールの公開 API を一箇所で管理できる（公開/非公開の制御が容易）
- サブパスの深い import を防げる（プロジェクトの規約に適合）
- 意図しない内部構造への依存を避けられる（変更に強い）

---

## 🔧 典型的な使い方（TypeScript 例）

ファイル構成:
```
components/
└─ profile-list/
   ├─ profile-list.tsx
   ├─ profile-list.logic.ts
   └─ index.ts   <-- バレル
```

`profile-list.tsx`:
```ts
// src/components/profile-list/profile-list.tsx
export const ProfileList = () => {
  return <div>プロフィール一覧</div>;
};

export type ProfileListProps = { /* ... */ };
```

`index.ts`（バレル/再エクスポート）:
```ts
// src/components/profile-list/index.ts
export { ProfileList } from "./profile-list";
export type { ProfileListProps } from "./profile-list";
export { someLogicFn } from "./profile-list.logic";
```

利用側:
```ts
import { ProfileList, someLogicFn } from "@/components/profile-list";
import type { ProfileListProps } from "@/components/profile-list";
```

---

## 📋 このプロジェクトにおけるルールとの関係
このリポジトリの `​.copilot-codeGeneration-instructions.md` の方針にあるルールに沿って、次の点を守ることが推奨されます。

- I-4: 各ディレクトリに必ず `index.ts` を設置し、そのレイヤーで公開してよいものだけを export（バレルを必須化）。
- I-5: `index.ts` は **再エクスポートのみ**。ロジックや関数定義を `index.ts` に書いてはいけない。
- I-6: `export default` 禁止 → 名前付きエクスポートだけを使う（`export const Foo` / `export { Foo }`）。
- I-3: サブパスインポート禁止 → バレルを使って `@/components/...` のような短いルートでインポートするのが推奨。
- 明示的なエクスポートを推奨（`export { X }`）、ワイルドカード（`export * from ...`）は安易に使わない方が安全。

---

## ⚠️ 注意点 / ベストプラクティス
- サイクル（循環依存）に注意：バレルを使うと見えにくい循環参照が発生しやすい（ESLint/型エラー・実行時エラーの原因）。
- `export * from` は便利だが名前衝突や意図しないエクスポート漏れを起こす可能性があるので、明示的な再エクスポート（`export { Foo } from './foo'`）が好ましい。
- 型のみ再エクスポートする場合は `export type { Foo } from './foo'` を使う（コンパイル時のみで副作用を避ける）。
- できるだけ副作用のある import を `index.ts` に置かない（`index.ts` は純粋に再エクスポートのみ）。
- Barrelを作ることで「公開API」を分かりやすくする一方、内部実装を隠すために `index.ts` に出すものを慎重に選ぶ。

---

## 📝 まとめ（簡潔に）
- Barrel Export はフォルダ単位での `index.ts` による再エクスポートをまとめる仕組みです。
- インポートが短くなり、APIの公開管理がしやすくなります。
- このプロジェクトでは `index.ts` を必須にして、再エクスポートのみ・名前付きエクスポートのみ使う方針になっています。
- 注意点として循環依存・ワイルドカード再エクスポート・副作用の混入に気をつけて実装してください。

---

This is a [Next.js](https://nextjs.org) project bootstrapped with [`create-next-app`](https://nextjs.org/docs/app/api-reference/cli/create-next-app).

## Getting Started

First, run the development server:

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
# or
bun dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

You can start editing the page by modifying `app/page.tsx`. The page auto-updates as you edit the file.

This project uses [`next/font`](https://nextjs.org/docs/app/building-your-application/optimizing/fonts) to automatically optimize and load [Geist](https://vercel.com/font), a new font family for Vercel.

## Learn More

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.

You can check out [the Next.js GitHub repository](https://github.com/vercel/next.js) - your feedback and contributions are welcome!

## Deploy on Vercel

The easiest way to deploy your Next.js app is to use the [Vercel Platform](https://vercel.com/new?utm_medium=default-template&filter=next.js&utm_source=create-next-app&utm_campaign=create-next-app-readme) from the creators of Next.js.

Check out our [Next.js deployment documentation](https://nextjs.org/docs/app/building-your-application/deploying) for more details.
