# SELECT in SELECT - 解答と解説

サブクエリ（SELECT の中に SELECT を書く）の学習ノート

---

## サブクエリとは？

SELECT 文の中に、もう1つの SELECT 文を入れる書き方。
括弧の中の SELECT が先に実行されて、**1つの値**に置き換わる。

---

## 例題：イギリスとドイツの間の人口の国を見つける

```sql
SELECT name, population FROM world
WHERE population BETWEEN
  (SELECT population+1 FROM world WHERE name = 'United Kingdom')
  AND
  (SELECT population-1 FROM world WHERE name = 'Germany')
```

### 内側から理解する

**ステップ1**：サブクエリ①を実行
```sql
(SELECT population+1 FROM world WHERE name = 'United Kingdom')
-- 結果例：67,530,173（イギリスの人口 + 1）
```

**ステップ2**：サブクエリ②を実行
```sql
(SELECT population-1 FROM world WHERE name = 'Germany')
-- 結果例：83,369,842（ドイツの人口 - 1）
```

**ステップ3**：サブクエリが数値に置き換わる
```sql
SELECT name, population FROM world
WHERE population BETWEEN 67530173 AND 83369842
```

**なぜ +1 と -1 をするのか？**
- `BETWEEN` は両端を含む（以上・以下）
- +1/-1 することで、イギリスとドイツ自身を結果から除外する

---

## 例題：ドイツ比のパーセンテージ表示

```sql
SELECT name,
  CONCAT(
    CAST(
      ROUND(100*population/(SELECT population FROM world WHERE name = 'Germany'), 0)
    AS int),
  '%')
FROM world
WHERE continent = 'Europe'
```

### 内側から外側へ分解

**層1**：ドイツの人口を取得
```sql
(SELECT population FROM world WHERE name = 'Germany')
-- 結果：83369843
```

**層2**：パーセンテージを計算
```sql
100 * population / 83369843
-- 例（フランス）：100 × 67059887 ÷ 83369843 = 80.407...
```

**層3**：整数に丸める
```sql
ROUND(80.407, 0)
-- 結果：80
```

**層4**：整数型に変換
```sql
CAST(80 AS int)
-- 結果：80（整数型）
```

**層5**：パーセント記号を連結
```sql
CONCAT(80, '%')
-- 結果：'80%'
```

### 関数まとめ

| 関数 | 意味 | 例 |
|------|------|-----|
| `ROUND(X, 0)` | 整数に丸める | `ROUND(80.7, 0)` → `81` |
| `ROUND(X, 1)` | 小数第1位に丸める | `ROUND(80.73, 1)` → `80.7` |
| `CAST(X AS int)` | 整数型に変換 | `CAST(80.0 AS int)` → `80` |
| `CONCAT(A, B)` | 文字列を連結 | `CONCAT(80, '%')` → `'80%'` |

---

## ⚠️ サブクエリのルール

1. **サブクエリは必ず括弧 `()` で囲む**
2. **サブクエリは1つの値を返すこと**（複数行はエラー）
3. **サブクエリの中でも計算や関数が使える**

### よくあるミス

```sql
-- ❌ サブクエリが複数行を返す（エラー）
WHERE population > (SELECT population FROM world WHERE continent = 'Europe')

-- ✅ 1つの値を返すようにする
WHERE population > (SELECT population FROM world WHERE name = 'Germany')
```

---

*学習中... 問題が進んだら追加していく*
