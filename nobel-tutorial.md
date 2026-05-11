# SELECT from Nobel Tutorial - 解答と解説

SQLZoo Nobel Tutorial（全16問）の解答・解説まとめ

テーブル構造：
```sql
nobel(yr, subject, winner)
```

---

## 問題1：Winners from 1950
1950年のノーベル賞受賞者を表示

```sql
SELECT yr, subject, winner
FROM nobel
WHERE yr = 1950
```

**ポイント**：`WHERE` で条件を指定する基本形

---

## 問題2：1962 Literature
1962年の文学賞受賞者を表示

```sql
SELECT winner
FROM nobel
WHERE yr = 1962
AND subject = 'literature'
```

**ポイント**：`AND` で複数条件を組み合わせる

---

## 問題3：Albert Einstein
アインシュタインが受賞した年と分野を表示

```sql
SELECT yr, subject
FROM nobel
WHERE winner = 'Albert Einstein'
```

---

## 問題4：Recent Peace Prizes
2000年以降の平和賞受賞者を表示

```sql
SELECT winner
FROM nobel
WHERE subject = 'Peace'
AND yr >= 2000
```

**ポイント**：`>=` は「以上」の意味

---

## 問題5：Literature in the 1980's
1980年代の文学賞を全情報表示

```sql
SELECT yr, subject, winner
FROM nobel
WHERE subject = 'literature'
AND yr BETWEEN 1980 AND 1989
```

**ポイント**：`BETWEEN X AND Y` は X以上 Y以下（両端を含む）

---

## 問題6：Only Presidents
特定の大統領たちの受賞情報を表示

```sql
SELECT * FROM nobel
WHERE winner IN (
  'Theodore Roosevelt',
  'Thomas Woodrow Wilson',
  'Jimmy Carter',
  'Barack Obama'
)
```

**ポイント**：`IN (...)` で「このリストのどれかに一致」を指定

---

## 問題7：John
ファーストネームが John の受賞者を表示

```sql
SELECT winner FROM nobel
WHERE winner LIKE 'John %'
```

**ポイント**：
- `LIKE` はパターンマッチング
- `%` は「何でもいい（0文字以上）」のワイルドカード
- `'John %'` は「John + スペース + 何か」にマッチ

---

## 問題8：Chemistry and Physics from different years
1980年の物理学賞と1984年の化学賞を表示

```sql
SELECT *
FROM nobel
WHERE (subject = 'physics' AND yr = 1980)
OR (subject = 'chemistry' AND yr = 1984)
```

**ポイント**：`OR` と括弧で複雑な条件を組み合わせる

---

## 問題9：Exclude Chemists and Medics
1980年の受賞者から化学と医学を除外

```sql
SELECT *
FROM nobel
WHERE yr = 1980
AND subject NOT IN ('chemistry', 'medicine')
```

**ポイント**：`NOT IN` で「このリストに含まれないもの」を指定

---

## 問題10：Early Medicine, Late Literature
1910年以前の医学賞と2004年以降の文学賞を表示

```sql
SELECT *
FROM nobel
WHERE (subject = 'medicine' AND yr < 1910)
OR (subject = 'literature' AND yr >= 2004)
```

---

## 問題11（Harder）：Umlaut
PETER GRÜNBERG の受賞情報を表示

```sql
SELECT *
FROM nobel
WHERE winner = 'Peter Grünberg'
```

**ポイント**：ウムラウト（ü）などの特殊文字もそのまま使える

---

## 問題12（Harder）：Apostrophe
EUGENE O'NEILL の受賞情報を表示

```sql
SELECT *
FROM nobel
WHERE winner = 'Eugene O''Neill'
```

**ポイント**：
- シングルクォート `'` を文字列内で使うには `''`（2つ重ねる）
- `O'Neill` → `'Eugene O''Neill'`

---

## 問題13（Harder）：Knights of the realm ⭐
「Sir」で始まる受賞者を、新しい年順、同年なら名前順で表示

```sql
SELECT winner, yr, subject
FROM nobel
WHERE winner LIKE 'Sir%'
ORDER BY yr DESC, winner
```

**ポイント**：
- `LIKE 'Sir%'`：「Sir」で始まるすべての名前にマッチ
- `ORDER BY yr DESC`：年を降順（新しい順）
- `, winner`：同年内は名前の昇順（A→Z、デフォルト）
- `ORDER BY` は複数キーをカンマ区切りで指定可能

**ORDER BY の基本ルール**：
| 書き方 | 意味 |
|--------|------|
| `ORDER BY yr` | 昇順（小→大）デフォルト |
| `ORDER BY yr ASC` | 昇順（明示的） |
| `ORDER BY yr DESC` | 降順（大→小） |

---

## 問題14（Harder）：Chemistry and Physics last ⭐⭐
1984年の受賞者を表示。chemistry と physics を最後にまわす

### 解法A：IN を値として使う
```sql
SELECT winner, subject
FROM nobel
WHERE yr = 1984
ORDER BY subject IN ('physics', 'chemistry'), subject, winner
```

### 解法B：CASE WHEN を使う（推奨）
```sql
SELECT winner, subject
FROM nobel
WHERE yr = 1984
ORDER BY
  CASE WHEN subject IN ('physics', 'chemistry') THEN 1 ELSE 0 END,
  subject, winner
```

**ポイント**：

**CASE WHEN の仕組み**：
- SQLの「if/else」にあたる条件分岐
- `CASE WHEN 条件 THEN 値A ELSE 値B END`

**なぜこれで「最後にまわせる」のか**：
1. physics/chemistry → `1` を返す
2. それ以外 → `0` を返す
3. `ORDER BY` はデフォルト昇順（小→大）
4. `0`（その他）が先、`1`（physics/chemistry）が後

**出力イメージ**：
```
《0のグループ》先に表示
Economics  ...
Literature ...
Medicine   ...
Peace      ...

《1のグループ》最後に表示
Chemistry  ...
Physics    ...
```

**覚え方**：
- 先にきてほしいもの → 小さい数字（0）
- 後にまわしたいもの → 大きい数字（1）

---

## 💡 このセクションで学んだ重要概念まとめ

| 概念 | 構文 | 用途 |
|------|------|------|
| 条件指定 | `WHERE` | 行をフィルタリング |
| 複数条件 | `AND` / `OR` | 条件の組み合わせ |
| 範囲指定 | `BETWEEN X AND Y` | X以上Y以下 |
| リスト一致 | `IN (...)` | リスト内のどれかに一致 |
| リスト除外 | `NOT IN (...)` | リスト内のどれにも一致しない |
| パターン検索 | `LIKE 'X%'` | ワイルドカードでパターンマッチ |
| 並び替え | `ORDER BY` | デフォルトは昇順 |
| 降順 | `ORDER BY ... DESC` | 大→小、新→古 |
| 条件分岐 | `CASE WHEN ... THEN ... ELSE ... END` | SQLの if/else |
| 特殊文字 | `''`（2つ重ね） | シングルクォートのエスケープ |
