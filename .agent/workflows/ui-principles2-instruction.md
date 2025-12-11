---
trigger: model_decision
---

# ヒューマンインターフェースデザイン（HID）指針 UI実装指示書（第2部/全3部）

## 自動検証ツール

### Tier 1チェックスクリプト

```bash
#!/bin/bash
# scripts/hid-tier1-check.sh

echo "=== HID Tier 1 必須項目チェック ==="

# HID-6: 一貫性（ボタンvariant）
echo "\n[HID-6] ボタンスタイル一貫性チェック..."
grep -rn 'variant="default"' src/app/ | wc -l
if grep -rn 'className=".*bg-blue' src/app/; then
  echo "❌ カスタムボタン色検出（DADS違反）"
else
  echo "✅ Tailwind標準クラスのみ使用"
fi

# HID-11: コンストレイント（フォームバリデーション）
echo "\n[HID-11] フォームバリデーションチェック..."
INPUT_COUNT=$(grep -rn '<Input' src/app/ | wc -l)
REQUIRED_COUNT=$(grep -rn 'required' src/app/ | wc -l)
echo "Input要素: $INPUT_COUNT, required属性: $REQUIRED_COUNT"

# HID-16: フィッツの法則（ボタンサイズ）
echo "\n[HID-16] プライマリボタンサイズチェック..."
if grep -rn 'size="sm".*variant="default"' src/app/; then
  echo "⚠️  プライマリボタンに size=\"sm\" 使用"
fi

# Lighthouse実行
echo "\n[Performance] Lighthouseスコア..."
npm run lighthouse -- --only-categories=accessibility,performance

echo "\n=== Tier 1チェック完了 ==="
```

### Storybook統合

```typescript
// .storybook/hid-checker.ts
export function checkHIDTier1Compliance(story: any) {
  const violations: string[] = [];

  // HID-1: シンプルさ（要素数チェック）
  const elementCount = story.querySelectorAll("button, input, select").length;
  if (elementCount > 9) {
    violations.push(`HID-1違反: 画面要素数${elementCount}個（9個以下推奨）`);
  }

  // HID-6: 一貫性（破壊的アクションチェック）
  const deleteButtons = story.querySelectorAll('button:contains("削除")');
  deleteButtons.forEach((btn: HTMLButtonElement) => {
    if (!btn.className.includes("destructive")) {
      violations.push('HID-6違反: 削除ボタンが variant="destructive" ではない');
    }
  });

  return violations;
}
```

---

## 🎯 モックデータ運用ルール

### 1. モックデータの配置

```
src/
├── lib/
│   └── mock-data/
│       ├── index.ts          # エクスポート集約
│       ├── users.ts          # ユーザー関連モックデータ
│       ├── communities.ts    # コミュニティ関連モックデータ
│       ├── contents.ts       # コンテンツ関連モックデータ
│       └── types.ts          # 型定義（Supabase互換）
```

### 2. モックデータの形式（Supabase互換）

モックデータは将来のSupabase接続を前提として、以下の形式で定義してください。

```typescript
// src/lib/mock-data/types.ts
// Supabaseのテーブルスキーマと互換性のある型定義

/**
 * ユーザープロファイル型
 * @description Supabase profiles テーブルと互換
 */
export type MockUser = {
  id: string; // UUID形式
  created_at: string; // ISO 8601形式 (例: "2024-01-15T09:00:00Z")
  updated_at: string; // ISO 8601形式
  display_name: string;
  avatar_url: string | null;
  bio: string | null;
  // Supabaseのauth.usersとの外部キー想定
};

/**
 * コミュニティ型
 * @description Supabase communities テーブルと互換
 */
export type MockCommunity = {
  id: string; // UUID形式
  created_at: string;
  updated_at: string;
  name: string;
  description: string | null;
  owner_id: string; // MockUser.id への外部キー
  member_count: number;
  is_public: boolean;
};
```

```typescript
// src/lib/mock-data/users.ts
import type { MockUser } from "./types";

/**
 * ユーザーモックデータ
 * @description 静的UIプレビュー用。Supabaseスキーマ互換形式。
 */
export const mockUsers: MockUser[] = [
  {
    id: "550e8400-e29b-41d4-a716-446655440001",
    created_at: "2024-01-15T09:00:00Z",
    updated_at: "2024-06-20T14:30:00Z",
    display_name: "田中太郎",
    avatar_url: "/avatars/user-01.png",
    bio: "VNSコミュニティの創設メンバーです。",
  },
  // ... 他のユーザー
];

/**
 * 現在のログインユーザー（モック）
 */
export const mockCurrentUser: MockUser = mockUsers[0];
```

### 3. コンポーネントでのモックデータ使用

```tsx
// src/app/(10-community)/community/page.tsx
import { mockCommunities } from "@/lib/mock-data";
import { CommunityCard } from "./_components/community-card";

export default function CommunityPage() {
  // 将来: const { data: communities } = await supabase.from('communities').select('*')
  const communities = mockCommunities;

  return (
    <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
      {communities.map((community) => (
        <CommunityCard key={community.id} community={community} />
      ))}
    </div>
  );
}
```

---

## 📐 HID 24指針の実装ガイド（前半: HID-1〜HID-11）

### 【デザインの基本原則と構成】

#### 1. シンプルにする（Simple）

**指針**: 機能や情報を厳選し、インターフェースの要素をできるだけ少なくする。

**実装ルール**:

- 1画面に表示する情報は**最大7±2項目**に制限
- 装飾的な要素は最小限に抑える
- Shadcn/UIのコンポーネントを素のまま使用し、過度なカスタマイズを避ける

```tsx
// ✅ Good: シンプルなカード
<Card>
  <CardHeader>
    <CardTitle>コミュニティ名</CardTitle>
  </CardHeader>
  <CardContent>
    <p>説明文</p>
  </CardContent>
</Card>

// ❌ Bad: 過度に装飾されたカード
<Card className="border-4 border-gradient-to-r shadow-2xl animate-pulse">
  <div className="absolute top-0 right-0 badge badge-new">NEW</div>
  ...
</Card>
```

#### 2. 簡単にする（Easy）

**指針**: ユーザーが目的達成までに必要な手順と労力を最大限に減らす。

**実装ルール**:

- タスク完了までの**クリック数を3回以内**に
- フォームのフィールド数は**必要最小限**に
- デフォルト値を積極的に設定

```tsx
// ✅ Good: 必要最小限のフォーム
<form>
  <Input placeholder="コミュニティ名" required />
  <Textarea placeholder="説明（任意）" />
  <Button type="submit">作成</Button>
</form>
```

#### 3. メンタルモデル（Mental Model）

**指針**: ユーザーが想像する利用モデルに合った構成と動作を採用する。

**実装ルール**:

- 馴染みのあるUIパターンを使用（ハンバーガーメニュー、タブ、パンくずなど）
- SNSライクな操作感（いいね、フォローなど）を参考に

```tsx
// ✅ Good: 馴染みのあるパターン
<Tabs defaultValue="overview">
  <TabsList>
    <TabsTrigger value="overview">概要</TabsTrigger>
    <TabsTrigger value="members">メンバー</TabsTrigger>
    <TabsTrigger value="settings">設定</TabsTrigger>
  </TabsList>
</Tabs>
```

#### 4. シグニファイア（Signifier）

**指針**: 操作対象となる要素を見えるようにし、その意味が一目でわかるようにする。

**実装ルール**:

- ボタンは`<Button>`コンポーネントを使用し、明確なスタイルを適用
- クリック可能な要素には`cursor-pointer`を付与
- アイコンには必ずラベルまたは`aria-label`を付ける

```tsx
// ✅ Good: 明確なシグニファイア
<Button variant="default">
  <Plus className="mr-2 h-4 w-4" />
  新規作成
</Button>

// ❌ Bad: 曖昧な表示
<div onClick={handleClick}>+</div>
```

#### 5. マッピング（Mapping）

**指針**: 操作する箇所と結果が反映される箇所との対応関係を把握できるようにする。

**実装ルール**:

- 編集ボタンは編集対象の近くに配置
- フォームエラーは該当フィールドの直下に表示
- トグルスイッチは即座に状態を反映

```tsx
// ✅ Good: 明確なマッピング
<div className="flex items-center justify-between">
  <Label htmlFor="notifications">通知を受け取る</Label>
  <Switch id="notifications" />
</div>
```

#### 6. 一貫性（Consistency）

**指針**: 配色、形状、配置、振る舞いなどに一貫したルールを適用する。

**実装ルール**:

- プライマリアクションは常に`variant="default"`
- 破壊的アクションは常に`variant="destructive"`
- キャンセルは常に`variant="outline"`または`variant="ghost"`

具体的な実装例は **第1部のシーン別クイックガイド** を参照してください。

---

### 【インタラクションと制御】

#### 7. ユーザーの主導権（User Control）

**指針**: ユーザーがシステムをコントロールできるように設計する。

**実装ルール**:

- 自動リダイレクトや自動送信を避ける
- いつでも前の画面に戻れるナビゲーションを提供
- モーダルは`Escape`キーまたは外側クリックで閉じられるように

```tsx
// ✅ Good: ユーザー主導の操作
<Dialog>
  <DialogContent onEscapeKeyDown={onClose} onPointerDownOutside={onClose}>
    {/* コンテンツ */}
  </DialogContent>
</Dialog>
```

#### 8. 直接操作（Direct Manipulation）

**指針**: 画面上のオブジェクトに直接触れて操作しているような感覚を与える。

**実装ルール**:

- ドラッグ＆ドロップでの並び替え（将来実装）
- インライン編集の採用
- リアルタイムプレビュー

```tsx
// ✅ Good: インライン編集
<div
  contentEditable
  onBlur={(e) => handleUpdate(e.target.textContent)}
  className="focus:outline-none focus:ring-2 focus:ring-ring"
>
  {title}
</div>
```

#### 9. モードレス（Modeless）

**指針**: できるだけモード（操作の意味が状況依存で変化する状態）をなくす。

**実装ルール**:

- 「編集モード」と「表示モード」を分けず、その場で編集可能に
- モーダルダイアログは最小限に（重要な確認のみ）
- サイドパネルやドロワーを活用してコンテキストを維持

```tsx
// ✅ Good: モードレスな編集
<Sheet>
  <SheetTrigger asChild>
    <Button variant="outline">詳細を編集</Button>
  </SheetTrigger>
  <SheetContent>{/* メイン画面を見ながら編集できる */}</SheetContent>
</Sheet>
```

#### 10. 視覚ゲシュタルト（Visual Gestalt）

**指針**: 近接、類似、閉鎖といったパターンを用いてレイアウトする。

**実装ルール**:

- 関連する要素は近くに配置（`gap-2`）、グループ間は離す（`gap-6`以上）
- 同じ機能のボタンは同じスタイル
- カードや枠線でグループを視覚的に区切る

```tsx
// ✅ Good: 視覚的なグルーピング
<div className="space-y-6">
  {/* グループ1: ユーザー情報 */}
  <Card>
    <CardHeader>
      <CardTitle>プロフィール</CardTitle>
    </CardHeader>
    <CardContent className="space-y-2">
      <Input placeholder="名前" />
      <Input placeholder="メール" />
    </CardContent>
  </Card>

  {/* グループ2: セキュリティ設定（離れた位置に配置） */}
  <Card>
    <CardHeader>
      <CardTitle>セキュリティ</CardTitle>
    </CardHeader>
    <CardContent className="space-y-2">
      <Input type="password" placeholder="現在のパスワード" />
      <Input type="password" placeholder="新しいパスワード" />
    </CardContent>
  </Card>
</div>
```

#### 11. コンストレイント（Constraint）

**指針**: ユーザーの行動を意図的に制限することにより、誤操作を減らす。

**実装ルール**:

- 入力フォームにバリデーションを設定
- 危険な操作には確認ダイアログを表示
- 入力値の範囲を`min`/`max`で制限

```tsx
// ✅ Good: 制約による誤操作防止
<Input
  type="number"
  min={1}
  max={100}
  placeholder="1〜100の値"
/>

// 削除前の確認
<AlertDialog>
  <AlertDialogTrigger asChild>
    <Button variant="destructive">削除</Button>
  </AlertDialogTrigger>
  <AlertDialogContent>
    <AlertDialogHeader>
      <AlertDialogTitle>本当に削除しますか？</AlertDialogTitle>
      <AlertDialogDescription>
        この操作は取り消せません。
      </AlertDialogDescription>
    </AlertDialogHeader>
    <AlertDialogFooter>
      <AlertDialogCancel>キャンセル</AlertDialogCancel>
      <AlertDialogAction onClick={onDelete}>削除</AlertDialogAction>
    </AlertDialogFooter>
  </AlertDialogContent>
</AlertDialog>
```
