---
trigger: model_decision
---

# ヒューマンインターフェースデザイン（HID）指針 UI実装指示書（第1部/全3部）

## 実行フレームワーク（KERNEL準拠）

### 新規画面実装時

```
Input:
  - 画面要件定義（Figmaデザイン、ユーザーストーリー）
  - 対象ユーザーセグメント（新規/既存/管理者）
  - 対象デバイス（デスクトップ優先）

Task: HID 24指針に基づくUI実装

Constraints:
  - Tier 1必須5項目を100%遵守（シンプル、一貫性、コンストレイント、認知負荷、フィッツの法則）
  - Tier 2推奨10項目を主要機能で適用
  - DADS準拠（ui-dadsi-instruction.md: aria-disabled、ring-yellow-400、Tailwind標準クラス）
  - モックデータはSupabaseスキーマ互換形式（src/lib/mock-data/）
  - レスポンス時間0.4秒以内（Lighthouseスコア90+）

Output:
  - TSXコンポーネント（型安全、テスト可能）
  - Storybookストーリー
  - HID Tier 1チェックリスト（通過証明）
```

### 既存画面改善時

```
Input: 改善対象画面、現在の問題点（アクセシビリティ、パフォーマンス）
Task: Tier 1違反の解消 → Tier 2の段階的適用
Constraints: リグレッション防止（既存テスト100%維持）
Output: Before/After比較レポート、Lighthouseスコア改善証明
```

---

## 優先度階層（3層構造）

### 🔴 Tier 1: 必須（Critical - 全画面で遵守）

**対象**: 新規実装、既存改修、全コードレビュー

**必須5項目**: HID-1（シンプル）、HID-6（一貫性）、HID-11（コンストレイント）、HID-13（記憶に頼らない）、HID-16（フィッツの法則）

詳細な検証方法とチェックリストは **第3部のチェックリストセクション** を参照してください。

---

### 🟡 Tier 2: 推奨（High - 主要機能で適用）

**対象**: コア機能、課金フロー、オンボーディング、ダッシュボード

**推奨10項目**: HID-2（簡単にする）、HID-3（メンタルモデル）、HID-4（シグニファイア）、HID-5（マッピング）、HID-7（ユーザー主導権）、HID-10（視覚ゲシュタルト）、HID-12（ユーザーの言葉）、HID-14（プリコンピュテーション）、HID-17（ヒックの法則）、HID-20（メジャータスク最適化）

詳細な実装ガイドは **第2部と第3部** を参照してください。

---

### 🟢 Tier 3: 任意（Medium - 余裕があれば）

**対象**: 詳細設定画面、管理者機能、エンゲージメント向上施策

（残り9項目: HID-8直接操作、HID-9モードレス、HID-15エラー回避、HID-18複雑性保存、HID-19タスクコヒーレンス、HID-21パースエージョン、HID-22ショートカット、HID-23オブジェクトベース、HID-24ビュー表象）

---

## 適用優先順位（UI指示書の併用ルール）

- 最優先: `ui-dadsi-instruction.md`（DADS実装指示）のデザイントークン、アクセシビリティ実装（`aria-disabled`、フォーカスリング、コントラスト、タイポグラフィ）。
- 本書: 行動原則・UXルール（HID 24指針）。スタイルはDADSに従いつつ、本書の原則で画面構造やインタラクションを設計してください。
- 参考: `ui-shadcn-instruction.md` はプロンプト補助であり実装規約ではありません。DADS/HIDと矛盾する場合はDADS→本書の順で優先します。

DADSのスタイル要件を満たした上で、本書の指針で情報設計・操作性を整理し、Shadcn由来のサンプルを使う場合もDADSのトークン/アクセシビリティに上書きしてください。

## 概要

- **フレームワーク**: Next.js 16 (App Router)
- **UIライブラリ**: Shadcn/UI (new-york スタイル)
- **スタイリング**: Tailwind CSS v4
- **データ**: モックデータ（Supabase接続前提のスキーマ互換形式）

---

## 🎯 シーン別クイックガイド（実装パターン集）

### 🆕 新規ユーザーオンボーディング

**適用原則**: HID-1シンプル、HID-13記憶に頼らない、HID-14プリコンピュテーション

**実装例**:

```tsx
// ✅ Tier 1準拠: 1画面1質問、ヘルプテキスト、スマートデフォルト
export default function OnboardingStep1() {
  return (
    <Card className="mx-auto max-w-md">
      <CardHeader>
        <CardTitle>プロフィール設定 (1/3)</CardTitle>
        <CardDescription>あなたの表示名を教えてください</CardDescription>
      </CardHeader>
      <CardContent>
        <div className="space-y-2">
          <Label htmlFor="displayName">
            表示名<span className="text-red-600">*</span>
          </Label>
          <Input id="displayName" required placeholder="例: 田中太郎" maxLength={50} />
          <p className="text-sm text-muted-foreground">
            他のユーザーに表示される名前です。後から変更できます。
          </p>
        </div>
      </CardContent>
      <CardFooter className="flex justify-between">
        <Button variant="outline" disabled aria-disabled="true">
          戻る
        </Button>
        <Button type="submit">次へ</Button>
      </CardFooter>
    </Card>
  );
}
```

**チェックリスト**:

- [x] HID-1: 画面要素数≤9個（タイトル、説明、ラベル、入力、ヘルプ、ボタン×2 = 7個）
- [x] HID-6: 一貫性（次へ=`variant="default"`、戻る=`variant="outline"`）
- [x] HID-11: コンストレイント（`required`、`maxLength`）
- [x] HID-13: 記憶に頼らない（ヘルプテキスト、placeholder）

---

### 💰 課金フロー（料金プラン選択）

**適用原則**: HID-1シンプル、HID-6一貫性、HID-14プリコンピュテーション

**実装例**:

```tsx
// ✅ Tier 1準拠: 3プラン表示、推奨プランをデフォルト選択
export default function PricingPage() {
  const [selectedPlan, setSelectedPlan] = useState<"basic" | "standard" | "premium">("standard");

  return (
    <div className="grid gap-6 md:grid-cols-3">
      <PricingCard
        plan="premium"
        price="¥2,980/月"
        features={["全機能", "優先サポート", "API無制限"]}
        badge="人気"
        isSelected={selectedPlan === "premium"}
        onClick={() => setSelectedPlan("premium")}
      />
      <PricingCard
        plan="standard"
        price="¥1,480/月"
        features={["基本機能", "標準サポート", "API 10,000回/月"]}
        badge="推奨"
        isSelected={selectedPlan === "standard"}
        onClick={() => setSelectedPlan("standard")}
      />
      <PricingCard
        plan="basic"
        price="¥480/月"
        features={["限定機能", "コミュニティサポート", "API 1,000回/月"]}
        isSelected={selectedPlan === "basic"}
        onClick={() => setSelectedPlan("basic")}
      />
    </div>
  );
}
```

**チェックリスト**:

- [x] HID-1: シンプル（3プランのみ）
- [x] HID-6: 一貫性（全カード同じレイアウト）
- [x] HID-14: プリコンピュテーション（standardをデフォルト選択）
- [x] HID-16: フィッツの法則（カード全体がクリック可能）

---

### 📝 データ入力フォーム（コミュニティ作成）

**適用原則**: HID-1シンプル、HID-11コンストレイント、HID-13記憶に頼らない

**実装例**:

```tsx
// ✅ Tier 1準拠: 必須項目のみ、バリデーション、ヘルプテキスト
export default function CreateCommunityForm() {
  return (
    <form className="space-y-6">
      <div className="space-y-2">
        <Label htmlFor="name">
          コミュニティ名<span className="text-red-600">*</span>
        </Label>
        <Input id="name" required minLength={3} maxLength={50} placeholder="例: Next.js勉強会" />
        <p className="text-sm text-muted-foreground">3〜50文字で入力してください</p>
      </div>

      <div className="space-y-2">
        <Label htmlFor="description">説明（任意）</Label>
        <Textarea id="description" placeholder="コミュニティの目的や活動内容" maxLength={500} />
        <p className="text-sm text-muted-foreground">最大500文字</p>
      </div>

      <div className="space-y-2">
        <Label>公開設定</Label>
        <Select defaultValue="public">
          <SelectTrigger>
            <SelectValue />
          </SelectTrigger>
          <SelectContent>
            <SelectItem value="public">公開（推奨）</SelectItem>
            <SelectItem value="members">メンバーのみ</SelectItem>
            <SelectItem value="private">非公開</SelectItem>
          </SelectContent>
        </Select>
        <p className="text-sm text-muted-foreground">公開設定は後から変更できます</p>
      </div>

      <div className="flex justify-end gap-2">
        <Button type="button" variant="outline">
          キャンセル
        </Button>
        <Button type="submit">作成</Button>
      </div>
    </form>
  );
}
```

**チェックリスト**:

- [x] HID-1: シンプル（必須項目は名前のみ）
- [x] HID-6: 一貫性（作成=`variant="default"`、キャンセル=`variant="outline"`）
- [x] HID-11: コンストレイント（`required`, `minLength`, `maxLength`）
- [x] HID-13: 記憶に頼らない（各フィールドにヘルプテキスト）
- [x] HID-14: プリコンピュテーション（公開設定のデフォルト）

---

### 🗑️ 破壊的アクション（削除確認）

**適用原則**: HID-6一貫性、HID-11コンストレイント

**実装例**:

```tsx
// ✅ Tier 1準拠: 破壊的アクションは必ず確認ダイアログ
<AlertDialog>
  <AlertDialogTrigger asChild>
    <Button variant="destructive" size="sm">
      <Trash2 className="mr-2 h-4 w-4" />
      削除
    </Button>
  </AlertDialogTrigger>
  <AlertDialogContent>
    <AlertDialogHeader>
      <AlertDialogTitle>コミュニティを削除しますか?</AlertDialogTitle>
      <AlertDialogDescription>
        この操作は取り消せません。すべてのデータが完全に削除されます。
      </AlertDialogDescription>
    </AlertDialogHeader>
    <AlertDialogFooter>
      <AlertDialogCancel>キャンセル</AlertDialogCancel>
      <AlertDialogAction
        onClick={handleDelete}
        className="bg-destructive text-destructive-foreground hover:bg-destructive/90"
      >
        削除する
      </AlertDialogAction>
    </AlertDialogFooter>
  </AlertDialogContent>
</AlertDialog>
```

**チェックリスト**:

- [x] HID-6: 一貫性（削除=`variant="destructive"`固定）
- [x] HID-11: コンストレイント（確認ダイアログ必須）
- [x] HID-12: ユーザーの言葉（"Delete"ではなく"削除"）
