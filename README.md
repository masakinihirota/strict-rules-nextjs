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

## 📝 Barrel Export まとめ（簡潔に）

- Barrel Export はフォルダ単位での `index.ts` による再エクスポートをまとめる仕組みです。
- インポートが短くなり、APIの公開管理がしやすくなります。
- このプロジェクトでは `index.ts` を必須にして、再エクスポートのみ・名前付きエクスポートのみ使う方針になっています。
- 注意点として循環依存・ワイルドカード再エクスポート・副作用の混入に気をつけて実装してください。

---

## 指示書改善指示: AIに指示を出す際のプロンプト設計のベストプラクティス6選

成功するプロンプトには6つの共通パターンがあると結論。

✅ K: シンプルに
良い：「Redisキャッシングの技術チュートリアルを書いて」
悪い：「Redisについて何か書いて」
→ トークン70%減、応答3倍速

✅E: 検証可能に
良い：「ユニットテストをパスするコードを出力して」
悪い：「〇〇についてプログラム書いて」
→ 成功率 41% → 85%

✅R: 再現性を確保
良い：「Python 3.12、Pandas 2.2で書け」
悪い：「最新のベストプラクティスで書いて」
→ 30日後でも94%一致

✅N: スコープを狭く
良い：「CSVファイルをマージするスクリプトを書いて」
悪い：「データ処理スクリプト＋テスト＋ドキュメント全部書いて」
→ 満足度41% → 89%

✅E: 制約を明示
良い：「外部ライブラリ禁止、関数20行以内」
悪い：「Pythonコードで」
→ 不要出力91%減

✅L: 論理構造で
良い：「Input: 複数CSV（同列）、Task: マージ、Constraints: Pandasのみ、50行以内、Output: merged.csv」
悪い：「データファイルを効率化するスクリプト書いて」
→ 200行使えないコード → 37行初回成功

これらを適用した結果：
初回成功率 72% → 94%
トークン -58%
修正回数 3.2 → 0.4

AIへプロンプト投げる時、少しだけこれを意識すると劇的に結果が変わります。

### 参考

XユーザーのJさん: 「海外Redditで、LLMへ渡すプロンプト設計に関する超有益ポスト。 ポスト主はプロンプト分析に1000時間以上費やし、成功するプロンプトには6つの共通パターンがあると結論。 ✅ K: シンプルに 悪い：「Redisについて何か書いて」 良い：「Redisキャッシングの技術チュートリアルを書いて」 →」 / X
https://x.com/j_kun_ml/status/1998697951485403647

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
