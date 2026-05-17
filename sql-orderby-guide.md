# 📊 ORDER BY・LENGTH・FETCH FIRST 完全ガイド

> この1問から学べることをぜんぶまとめました。

---

## 元の問題

> STATION テーブルから、CITY 名が **最短** と **最長** の都市を、  
> 文字数と一緒に表示せよ。  
> 同じ長さの都市が複数ある場合は **アルファベット順で先のもの** を選べ。

### 答え

```sql
-- 最短
SELECT CITY, LENGTH(CITY)
FROM STATION
ORDER BY LENGTH(CITY), CITY
FETCH FIRST 1 ROW ONLY;

-- 最長
SELECT CITY, LENGTH(CITY)
FROM STATION
ORDER BY LENGTH(CITY) DESC, CITY
FETCH FIRST 1 ROW ONLY;
```

---

## 🔤 ORDER BY（並べ替え）

### 基本：昇順（小さい順）

```sql
SELECT * FROM STATION ORDER BY CITY;
```

```
結果：
ABC
DEF
New York
Tokyo
```

何も指定しないと **ASC（昇順）** になる。アルファベット順、数字なら小さい順。

---

### DESC をつけると降順（大きい順）

```sql
SELECT * FROM STATION ORDER BY CITY DESC;
```

```
結果：
Tokyo
New York
DEF
ABC
```

---

### 数字でも使える

```sql
SELECT * FROM STATION ORDER BY POPULATION;       -- 人口少ない順
SELECT * FROM STATION ORDER BY POPULATION DESC;   -- 人口多い順
```

---

### 🔥 2段階ソート（超重要！）

```sql
ORDER BY LENGTH(CITY), CITY
```

これは **「まず文字数で並べて、文字数が同じならアルファベット順で並べて」** という意味。

```
元データ：
CITY         文字数
──────────────────
DEF           3
ABC           3    ← DEF と同じ長さ！
New York      8
Los Angeles  11

ORDER BY LENGTH(CITY), CITY の結果：
CITY         文字数
──────────────────
ABC           3    ← 同じ3文字の中でアルファベット順が先
DEF           3
New York      8
Los Angeles  11
```

**カンマで区切って複数の条件を書ける。左が優先。**

```sql
-- 書き方のパターン
ORDER BY 第1条件, 第2条件
ORDER BY 第1条件 DESC, 第2条件
ORDER BY 第1条件, 第2条件 DESC
ORDER BY 第1条件 DESC, 第2条件 DESC
```

> 💡 DESC は **それぞれの列に個別につける**。  
> `ORDER BY LENGTH(CITY) DESC, CITY` だと  
> 文字数は **長い順**、同じ長さの中ではアルファベットは **ABC順**。

---

## 📏 LENGTH（文字数を数える）

```sql
SELECT CITY, LENGTH(CITY) FROM STATION;
```

```
CITY          LENGTH(CITY)
───────────────────────────
ABC           3
New York      8
Los Angeles   11
San Francisco 13
```

やってることは **文字数を数えるだけ**。スペースも1文字としてカウントされる。

### SELECT にも WHERE にも ORDER BY にも使える

```sql
-- 表示に使う
SELECT CITY, LENGTH(CITY) FROM STATION;

-- 絞り込みに使う
SELECT CITY FROM STATION WHERE LENGTH(CITY) > 10;

-- 並べ替えに使う
SELECT CITY FROM STATION ORDER BY LENGTH(CITY);
```

---

## 🎯 FETCH FIRST / LIMIT（上から○件取る）

### Oracle の場合

```sql
FETCH FIRST 1 ROW ONLY     -- 1件だけ
FETCH FIRST 5 ROWS ONLY    -- 5件だけ
FETCH FIRST 10 ROWS ONLY   -- 10件だけ
```

### MySQL の場合

```sql
LIMIT 1      -- 1件だけ
LIMIT 5      -- 5件だけ
LIMIT 10     -- 10件だけ
```

### 意味は同じ

```
並べ替え後：
1位  ABC           ← FETCH FIRST 1 → ここだけ取る
2位  DEF
3位  New York
4位  Los Angeles

FETCH FIRST 3 なら：
1位  ABC           ← 取る
2位  DEF           ← 取る
3位  New York      ← 取る
4位  Los Angeles
```

---

## 🧩 全部組み合わせるとこうなる

この問題の SQL を **日本語に翻訳** するとこう：

```sql
SELECT CITY, LENGTH(CITY)    -- 都市名と文字数を見せて
FROM STATION                 -- STATION テーブルから
ORDER BY LENGTH(CITY), CITY  -- 文字数が短い順に、同じなら ABC 順に並べて
FETCH FIRST 1 ROW ONLY;      -- 一番上の1件だけちょうだい
```

```
ステップ1：全データ取得（FROM）
  DEF(3), ABC(3), New York(8), Los Angeles(11)

ステップ2：並べ替え（ORDER BY）
  ABC(3), DEF(3), New York(8), Los Angeles(11)

ステップ3：先頭だけ取る（FETCH FIRST）
  ABC(3)  ← これが答え！
```

---

## 📋 まとめ表

| やりたいこと | 書き方 | 例 |
|------------|--------|-----|
| 昇順に並べる | `ORDER BY 列` | `ORDER BY CITY` |
| 降順に並べる | `ORDER BY 列 DESC` | `ORDER BY CITY DESC` |
| 2段階で並べる | `ORDER BY 列1, 列2` | `ORDER BY LENGTH(CITY), CITY` |
| 文字数を数える | `LENGTH(列)` | `LENGTH(CITY)` |
| 上から1件取る | `FETCH FIRST 1 ROW ONLY` | （Oracle） |
| 上から1件取る | `LIMIT 1` | （MySQL） |

---

## 💡 ORDER BY を使う実践パターン

```sql
-- 売上トップ3
SELECT * FROM sales ORDER BY revenue DESC FETCH FIRST 3 ROWS ONLY;

-- 最新の注文5件
SELECT * FROM orders ORDER BY order_date DESC LIMIT 5;

-- 一番古いユーザー
SELECT * FROM users ORDER BY created_at FETCH FIRST 1 ROW ONLY;

-- 名前が一番長い商品
SELECT name, LENGTH(name) FROM products ORDER BY LENGTH(name) DESC LIMIT 1;
```

---

## 🔗 SQL の実行順序（おまけ）

SQL は書いた順番じゃなくて、この順番で動く：

```
1. FROM     ← まずテーブルを選ぶ
2. WHERE    ← 条件で絞る
3. SELECT   ← 表示する列を選ぶ
4. ORDER BY ← 並べ替える
5. FETCH / LIMIT ← 件数を制限する
```

だから ORDER BY は SELECT の **後** に実行される。  
FETCH / LIMIT は **一番最後** に実行される。

---

*この1問だけでこれだけ学べる。SQL は積み重ねだ！* 💪
