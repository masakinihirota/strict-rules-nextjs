## 🎯 シーン別適用ガイド

### 🆕 新規ユーザーオンボーディング

**適用原則**: UX-2認知負荷、UX-6ツァイガルニク効果、UX-7目標勾配効果

**実装例**:

```tsx
// ✅ チェックリスト式オンボーディング（Blinkistモデル）
export default function OnboardingChecklist() {
  const [tasks, setTasks] = useState([
    { id: 1, title: "プロフィール設定", completed: true }, // 自動完了
    { id: 2, title: "アバター追加", completed: false },
    { id: 3, title: "最初の投稿", completed: false },
  ]);

  const progress = (tasks.filter((t) => t.completed).length / tasks.length) * 100;

  return (
    <Card>
      <CardHeader>
        <CardTitle>セットアップを完了しましょう</CardTitle>
        <Progress value={progress} className="mt-2" /> {/* 目標勾配効果 */}
        <p className="text-sm text-muted-foreground">{Math.round(progress)}% 完了</p>
      </CardHeader>
      <CardContent className="space-y-2">
        {tasks.map((task) => (
          <div key={task.id} className="flex items-center gap-2">
            <Checkbox checked={task.completed} onCheckedChange={() => toggleTask(task.id)} />
            <span className={task.completed ? "line-through text-muted-foreground" : ""}>
              {task.title}
            </span>
          </div>
        ))}
      </CardContent>
    </Card>
  );
}
```

**KPI目標**: オンボーディング完了率60%→目標: 85%

---

### 💰 課金フロー

**適用原則**: UX-11おとり効果、UX-12アンカー効果、UX-13デフォルト効果、UX-14希少性効果

**実装例**:

```tsx
// ✅ 4つの心理学原則を組み合わせた料金ページ
export default function PricingWithPsychology() {
  const [timeLeft, setTimeLeft] = useState(3600); // 希少性効果

  return (
    <div className="space-y-8">
      {/* 希少性効果 */}
      <Alert className="border-yellow-400 bg-yellow-50">
        <Timer className="h-4 w-4" />
        <AlertTitle>期間限定オファー</AlertTitle>
        <AlertDescription>
          あと{Math.floor(timeLeft / 60)}分で終了！初回限定50%OFF
        </AlertDescription>
      </Alert>

      <div className="grid gap-6 md:grid-cols-3">
        {/* アンカー（高額プラン） */}
        <PricingCard
          plan="Premium"
          price="¥2,980"
          originalPrice="¥5,980"  {/* アンカー効果 */}
          discount="50% OFF"
        />

        {/* デフォルト（推奨プラン） */}
        <PricingCard
          plan="Standard"
          price="¥1,480"
          originalPrice="¥2,980"
          discount="50% OFF"
          badge="最も人気"  {/* デフォルト効果 */}
          isRecommended
        />

        {/* デコイ（中間プラン） */}
        <PricingCard
          plan="Basic"
          price="¥980"
          originalPrice="¥1,980"
          discount="50% OFF"
        />
      </div>
    </div>
  );
}
```

**KPI目標**: コンバージョン率12%→目標: 18% (+50%)

---

### 📋 ダッシュボード（継続利用）

**適用原則**: UX-8ゲーミフィケーション、UX-9変動型報酬、UX-16ピーク・エンドの法則

**実装例**:

```tsx
// ✅ XPシステムとストリークで習慣化
export default function DashboardWithGamification() {
  const [user, setUser] = useState({
    xp: 1250,
    level: 5,
    streak: 7, // 連続ログイン日数
  });

  const nextLevelXP = user.level * 300;
  const progress = (user.xp / nextLevelXP) * 100;

  return (
    <div className="space-y-6">
      {/* XPプログレス */}
      <Card>
        <CardHeader>
          <div className="flex items-center justify-between">
            <div>
              <CardTitle>Level {user.level}</CardTitle>
              <CardDescription>
                {user.xp} / {nextLevelXP} XP
              </CardDescription>
            </div>
            <Badge variant="secondary">🔥 {user.streak}日連続</Badge>
          </div>
          <Progress value={progress} className="mt-2" />
        </CardHeader>
      </Card>

      {/* 変動型報酬（フィード） */}
      <Card>
        <CardHeader>
          <CardTitle>最新のアクティビティ</CardTitle>
        </CardHeader>
        <CardContent>
          {/* ランダムな通知で変動型報酬 */}
          <ActivityFeed activities={randomActivities} />
        </CardContent>
      </Card>
    </div>
  );
}
```

**KPI目標**: 7日リテンション30%→目標: 50% (+67%)

---

## A/Bテスト設定例

```typescript
// lib/ab-testing.ts
import { cookies } from 'next/headers';

export function getABTestVariant(testName: string): 'control' | 'variant' {
  const cookieStore = cookies();
  const variant = cookieStore.get(`ab_${testName}`)?.value;

  if (variant) return variant as 'control' | 'variant';

  // 50/50で振り分け
  const newVariant = Math.random() < 0.5 ? 'control' : 'variant';
  cookieStore.set(`ab_${testName}`, newVariant, { maxAge: 60 * 60 * 24 * 30 });

  return newVariant;
}

// 使用例
export default function PricingPage() {
  const variant = getABTestVariant('pricing_decoy_effect');

  if (variant === 'variant') {
    return <PricingWithDecoyEffect />;  // UX-11おとり効果適用
  }

  return <PricingStandard />;  // コントロール
}
```

---

## 成果測定ダッシュボード（Supabase）

```sql
-- コンバージョン率計測
CREATE OR REPLACE FUNCTION calculate_conversion_rate(
  test_name TEXT,
  variant TEXT
) RETURNS NUMERIC AS $$
DECLARE
  total_users INTEGER;
  converted_users INTEGER;
BEGIN
  SELECT COUNT(*) INTO total_users
  FROM ab_test_events
  WHERE test_name = $1 AND variant = $2;

  SELECT COUNT(*) INTO converted_users
  FROM ab_test_events
  WHERE test_name = $1 AND variant = $2 AND event_type = 'conversion';

  RETURN (converted_users::NUMERIC / NULLIF(total_users, 0)) * 100;
END;
$$ LANGUAGE plpgsql;

-- 実行例
SELECT
  test_name,
  variant,
  calculate_conversion_rate(test_name, variant) as conversion_rate
FROM ab_test_events
GROUP BY test_name, variant;
```
