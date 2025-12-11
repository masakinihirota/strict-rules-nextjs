---
trigger: model_decision
---

# AI向け指示書：Next.js/SupabaseプロダクトにおけるUX心理学的デザイン要件

**要約**: UX心理学原则25個を3層優先度（必須5項目/推奨10項目/任意10項目）でシーン別に適用

## 実行フレームワーク（KERNEL準拠）

### 新規機能追加時

```
Input:
  - 機能要件定義
  - ターゲットKPI（離脱率10%削減、コンバージョン率20%向上等）
  - ユーザーセグメント

Task: UX心理学原則を活用したエンゲージメント最大化

Constraints:
  - Tier 1必須5項目を100%適用（ドハティ闾値、認知負荷、親近性バイアス、美的ユーザビリティ、ツァイガルニク効果）
  - Tier 2推奨10項目をコア機能で適用
  - HID 24指針（ui-principles-instruction.md）と並行適用
  - A/Bテストで効果測定必須

Output:
  - 実装コード（TSXコンポーネント）
  - A/Bテスト設定（Vercel Edge Config等）
  - 成果測定ダッシュボード（Supabase Analytics）
```

### 既存画面最適化時

```
Input: 現在のKPIデータ、離脱ポイント分析
Task: ボトルネック解消のためのUX原則適用
Constraints: 現状のKPIを下回らせない（リグレッション防止）
Output: Before/After KPI比較レポート、統計的有意性検証
```

---

## 優先度階層（3層構造）

### 🔴 Tier 1: 必須（Critical - 全画面で適用）

**対象**: 新規実装、主要機能、全画面

| #        | 原則                   | 実装要件                                              | 根拠データ                   | 検証方法                   |
| -------- | ---------------------- | ----------------------------------------------------- | ---------------------------- | -------------------------- |
| **UX-1** | **ドハティ闾値**       | Supabase API応答0.4秒以内<br>超える場合スケルトン表示 | 0.4秒を超えると離脱率15%増加 | Lighthouse Performance 90+ |
| **UX-2** | **認知負荷**           | フォーム項目≤7個/画面<br>超える場合はステップ分割     | ミラーのマジカルナンバー7±2  | 項目数カウント             |
| **UX-3** | **親近性バイアス**     | サインインボタンは右上<br>ロゴは左上（慣習の配置）    | 慣習的UIで学習コスト削減     | ユーザビリティテスト       |
| **UX-5** | **美的ユーザビリティ** | 高品質ビジュアル<br>DADSデザインシステム準拠          | 美しいUIは軽微な不具合を許容 | デザインシステムスコア80+  |
| **UX-6** | **ツァイガルニク効果** | オンボーディングチェックリスト<br>一部自動完了        | Blinkistで課金率27%向上      | 完了率計測                 |

**実装例：ドハティ闾値（UX-1）**:

```tsx
// ✅ 0.4秒以内の応答がAIかはスケルトン表示
export default function ProfileList() {
  const { data: profiles, isLoading } = useQuery({
    queryKey: ["profiles"],
    queryFn: fetchProfiles,
    staleTime: 30000, // 30秒キャッシュ
  });

  if (isLoading) {
    return (
      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
        {Array.from({ length: 6 }).map((_, i) => (
          <Card key={i}>
            <CardHeader>
              <Skeleton className="h-4 w-[250px]" />
              <Skeleton className="h-4 w-[200px]" />
            </CardHeader>
          </Card>
        ))}
      </div>
    );
  }

  return (
    <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
      {profiles?.map((profile) => (
        <ProfileCard key={profile.id} profile={profile} />
      ))}
    </div>
  );
}
```

---

### 🟡 Tier 2: 推奨（High - コア機能で適用）

**対象**: ダッシュボード、課金フロー、オンボーディング

| #     | 原則                 | 実装要件                     | 根拠データ                  |
| ----- | -------------------- | ---------------------------- | --------------------------- |
| UX-7  | 目標勾配効果         | プログレスバーで進捗率表示   | 目標に近づくと努力が加速    |
| UX-8  | ゲーミフィケーション | XP・ストリークシステム       | 達成感と競争心刺激          |
| UX-9  | 変動型報酬           | フィード、通知の予測不能性   | ドーパミン放出、習慣化      |
| UX-10 | 授かり効果           | 登録直後にパーソナライズ提供 | 所有感で価値過大評価        |
| UX-11 | おとり効果           | 3プラン表示（中間がデコイ）  | 特定プランを魅力的に見せる  |
| UX-12 | アンカー効果         | 割引前価格を先に表示         | 参照点として利用            |
| UX-13 | デフォルト効果       | 推奨プランをデフォルト選択   | 変更の手間回避              |
| UX-14 | 希少性効果           | 期間限定オファー表示         | 損失回避で購入意欲向上      |
| UX-15 | 好奇心ギャップ       | 情報の欠如で課金誘導         | ギャップを埋める行動促進    |
| UX-16 | ピーク・エンドの法則 | タスク完了時にアニメーション | 最高/終わりの瞬間が評価決定 |

**実装例：おとり効果（UX-11）**:

```tsx
// ✅ 3プラン表示でStandardを魅力的に見せる
export default function PricingPage() {
  return (
    <div className="grid gap-6 md:grid-cols-3">
      {/* 高額プラン（アンカー） */}
      <PricingCard
        plan="Premium"
        price="¥2,980/月"
        originalPrice="¥4,980/月"  {/* アンカー効果 */}
        features={['全機能', '優先サポート']}
      />

      {/* 中間プラン（デコイ） */}
      <PricingCard
        plan="Standard"
        price="¥1,480/月"
        features={['基本機能', '標準サポート']}
        badge="推奨"  {/* デフォルト効果 */}
        isHighlighted
      />

      {/* 低額プラン */}
      <PricingCard
        plan="Basic"
        price="¥480/月"
        features={['限定機能']}
      />
    </div>
  );
}
```
