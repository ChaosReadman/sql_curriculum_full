# Day10_実用SQL問題

---

## 🎯 目次
- [問題1: 複数条件でのデータ集計](#問題1-複数条件でのデータ集計)
- [問題2: ウィンドウ関数を使った最新レコードの取得](#問題2-ウィンドウ関数を使った最新レコードの取得)
- [問題3: 存在しないレコードを条件にしたデータ更新](#問題3-存在しないレコードを条件にしたデータ更新)
- [✅ 振り返りチェック](#-振り返りチェック)

---

これまでの学習内容を総動員して、より実践的なSQL問題に挑戦しましょう。

## 問題1: 複数条件でのデータ集計
`JOIN`, `WHERE`, `GROUP BY`, `HAVING` を組み合わせた、集計クエリの基本形です。

### 問題
2025年7月中に、商品を**2つ以上**購入した**レジ操作者**の名前と、その期間内の**売上合計金額**、**売上件数**を、合計金額の降順で表示してください。

### 解答
```sql
SELECT
    ru.name AS RegisterUserName,
    SUM(rs.sale_amount) AS TotalSales,
    COUNT(rs.id) AS SalesCount
FROM
    register_sales AS rs
JOIN
    register_users AS ru ON rs.register_user_id = ru.id
WHERE
    rs.sale_date >= '2025-07-01' AND rs.sale_date < '2025-08-01'
GROUP BY
    ru.name
HAVING
    COUNT(rs.id) >= 2
ORDER BY
    TotalSales DESC;
```

### 解説
1.  **`FROM` & `JOIN`**: `register_sales`（売上）と`register_users`（操作者）を`register_user_id`で結合し、操作者名を取得できるようにします。
2.  **`WHERE`**: `sale_date`で2025年7月1日から7月31日までのデータに絞り込みます。集計前のフィルタリングです。
3.  **`GROUP BY`**: `ru.name`（操作者名）でグループ化し、操作者ごとの集計を準備します。
4.  **`SELECT`**: `SUM`で売上合計を、`COUNT`で売上件数を計算します。
5.  **`HAVING`**: `GROUP BY`で集計した結果に対し、「売上件数が2件以上」という条件でさらに絞り込みます。
6.  **`ORDER BY`**: 最終的な結果を`TotalSales`（売上合計）の降順で並び替えます。

### 注意点
*   **`WHERE`と`HAVING`の違い**を明確に意識しましょう。`WHERE`は集計前（`GROUP BY`の前）の個々の行に対する条件、`HAVING`は集計後（`GROUP BY`の後）のグループに対する条件です。
*   日付の範囲指定で `BETWEEN '2025-07-01' AND '2025-07-31'` も使えますが、`sale_date`が`DATETIME`型の場合、`'2025-07-31 00:00:00'`までしか含まれません。`>=`と`<`の組み合わせは、時刻まで考慮した安全な範囲指定としてよく使われます。

---

## 問題2: ウィンドウ関数を使った最新レコードの取得
`GROUP BY`では難しい「各グループの最新N件」といった種類の問い合わせは、ウィンドウ関数が得意とするところです。

### 問題
各商品（書籍または文房具）が**最後に売れた日**と、その時の**レジ担当者名**を表示してください。

### 解答
```sql
WITH RankedSales AS (
    SELECT
        COALESCE(b.title, s.name) AS ProductName,
        rs.sale_date,
        ru.name AS RegisterUserName,
        ROW_NUMBER() OVER(PARTITION BY ps.id ORDER BY rs.sale_date DESC) AS rn
    FROM
        register_sales AS rs
    JOIN
        product_stocks AS ps ON rs.product_stock_id = ps.id
    JOIN
        register_users AS ru ON rs.register_user_id = ru.id
    LEFT JOIN
        books AS b ON ps.book_id = b.id
    LEFT JOIN
        stationery AS s ON ps.stationery_id = s.id
)
SELECT
    ProductName,
    sale_date AS LastSaleDate,
    RegisterUserName
FROM
    RankedSales
WHERE
    rn = 1
ORDER BY
    ProductName;
```

### 解説
1.  **`WITH ... AS (...)`**: CTE（共通テーブル式）を使い、`RankedSales`という名前の一時的な結果セットを定義します。クエリが読みやすくなります。
2.  **`ROW_NUMBER() OVER(...)`**: ウィンドウ関数を使い、各行に順位を付けます。
    *   `PARTITION BY ps.id`: `product_stocks`のID（＝商品）ごとにデータをグループ分けします。
    *   `ORDER BY rs.sale_date DESC`: 各グループ内で、販売日が新しい順に並べ替えます。
    *   `ROW_NUMBER()`は、この並び順に従って `1, 2, 3, ...` と連番を振ります。つまり、各商品で最も新しい売上が `rn = 1` となります。
3.  **最後の`SELECT`**: CTE `RankedSales`から、`WHERE rn = 1` の条件で「各商品の最新の売上」レコードだけを抽出します。

---

## 問題3: 存在しないレコードを条件にしたデータ更新
`UPDATE`文と`NOT EXISTS`を組み合わせることで、複雑な条件でのデータ更新が可能です。

### 問題
書籍の中で、**一度も売れたことがない**書籍の価格を、現在の価格から100円引きにしてください。ただし、価格が100円未満になる場合は0円とします。

### 解答
```sql
UPDATE b
SET
    price = CASE
                WHEN b.price > 100 THEN b.price - 100
                ELSE 0
            END
FROM
    books AS b
WHERE
    NOT EXISTS (
        SELECT 1
        FROM product_stocks AS ps
        JOIN register_sales AS rs ON ps.id = rs.product_stock_id
        WHERE ps.book_id = b.id
    );
```

### 解説
1.  **`UPDATE b ... FROM books AS b`**: 更新対象の`books`テーブルに`b`というエイリアス（別名）を付けています。
2.  **`WHERE NOT EXISTS (...)`**: `EXISTS`は、括弧内の副問い合わせが1行でも結果を返せば`TRUE`、1行も返さなければ`FALSE`になります。`NOT EXISTS`はその逆で、「副問い合わせの結果が1行も存在しない」場合に`TRUE`となります。
3.  **副問い合わせ**: `books`テーブルの各行（`b.id`）に対して、「その本が売れた記録（`register_sales`テーブルに存在するレコード）」を探しています。売れた記録がなければ、この副問い合わせは0行を返し、`NOT EXISTS`が`TRUE`となって`UPDATE`の対象になります。
4.  **`CASE`式**: `SET`句の中で条件分岐を行っています。価格が100円より大きい場合は100を引き、そうでなければ0を設定します。

---

## ✅ 振り返りチェック

- [ ] `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY` を組み合わせた集計クエリを記述できる
- [ ] ウィンドウ関数（`ROW_NUMBER`など）を使って、グループ内の最新レコードを取得する方法を説明できる
- [ ] `NOT EXISTS` を使って、関連テーブルに存在しないレコードを更新対象とする方法を説明できる
- [ ] `CASE`式を使って、条件に応じた値の設定ができる

---
