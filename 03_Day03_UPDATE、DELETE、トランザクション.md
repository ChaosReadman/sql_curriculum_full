# 03_Day03_UPDATE、DELETE、トランザクション

---

## 🎯 目次
- [🎯 目次](#-目次)
- [📝 内容詳細](#-内容詳細)
   - [1. UPDATE文の基本構文を理解し、実行してみましょう。](#1-update文の基本構文を理解し実行してみましょう)
   - [2. 書籍IDが1（＝『AIの未来』）の価格を10%アップします。](#2-書籍idが1＝aiの未来の価格を10%アップします)
   - [3. 在庫数を補充（文房具ID = 2 の在庫を+20）](#3-在庫数を補充文房具id--2-の在庫を20)
   - [4. 田中太郎のメールアドレスを変更](#4-田中太郎のメールアドレスを変更)
   - [5. DELETE文の基本構文を理解し、実行してみましょう。](#5-delete文の基本構文を理解し実行してみましょう)
   - [6. 古い売上データを削除（2025年7月1日以前）](#6-古い売上データを削除2025年7月1日以前)
   - [7. 特定のレジ操作者のデータ削除（花子）](#7-特定のレジ操作者のデータ削除花子)
   - [8. 売れた商品は在庫を1つ減らす（仮定：売上1件ごとに1個売れた）](#8-売れた商品は在庫を1つ減らす仮定売上1件ごとに1個売れた)
   - [9. トランザクションの基本を理解し、実行してみましょう。](#9-トランザクションの基本を理解し実行してみましょう)
   - [10. トランザクションを使って、書籍の価格更新と在庫補充を同時に行います。](#10-トランザクションを使って書籍の価格更新と在庫補充を同時に行います)
   - [11. トランザクションのロールバックを実行してみましょう。](#11-トランザクションのロールバックを実行してみましょう)
- [✅ 振り返りチェック](#-振り返りチェック)


---

## 📝 内容詳細

## 1. UPDATE文の基本構文を理解し、実行してみましょう。

```sql
UPDATE テーブル名
SET カラム名 = 新しい値
WHERE 条件;
```

## 2. 書籍IDが1（＝『AIの未来』）の価格を10%アップします。
``` sql
UPDATE books
SET price = price * 1.1
WHERE id = 1;
```

## 3. 在庫数を補充（文房具ID = 2 の在庫を+20）
```sql
-- 文房具IDが2の在庫数を20増やします。
UPDATE product_stocks
SET quantity = quantity + 20
WHERE stationery_id = 2;
```

## 4. 田中太郎のメールアドレスを変更
```sql
-- 田中太郎のメールアドレスを`t.taro@example.com`に変更します。
UPDATE register_users
SET email = 't.taro@example.com'
WHERE name = '田中 太郎';
```

## 5. DELETE文の基本構文を理解し、実行してみましょう。
```sql
--DELETE FROM テーブル名
--WHERE 条件;
```

## 6. 古い売上データを削除（2025年7月1日以前）
```sql
DELETE FROM register_sales
WHERE sale_date < '2025-07-01';
```

## 7. 特定のレジ操作者のデータ削除（花子）
```sql
DELETE FROM register_users
WHERE name = '佐藤 花子';
```
 - ただし、このSQLはエラーになります。
 - なぜなら、`register_users` テーブルは `register_sales` テーブルと外部キー制約で結ばれているため、`register_users` のレコードを削除する前に、`register_sales` の関連レコードを削除する必要があります。
 
## 8. 売れた商品は在庫を1つ減らす（仮定：売上1件ごとに1個売れた）
```sql
-- 売上があった商品在庫を1つ減らします。
UPDATE product_stocks
SET quantity = quantity - 1
WHERE id IN (
    SELECT product_stock_id
    FROM register_sales
    WHERE sale_date >= '2025-07-01'
);
```

## 9. トランザクションの基本を理解し、実行してみましょう。

```sql
BEGIN TRANSACTION;
-- ここにトランザクション内で実行するSQLを記述
COMMIT; -- または ROLLBACK;
```
## 10. トランザクションを使って、書籍の価格更新と在庫補充を同時に行います。

```sql
BEGIN TRANSACTION;
-- 書籍の価格を更新
UPDATE books
SET price = price * 1.1
WHERE id = 1;
-- 文房具の在庫を補充
UPDATE product_stocks
SET quantity = quantity + 20
WHERE stationery_id = 2;
COMMIT; -- すべて成功したらコミット
```
## 11. トランザクションのロールバックを実行してみましょう。

```sql
BEGIN TRANSACTION;
-- 書籍の価格を更新
UPDATE books
SET price = price * 1.1
WHERE id = 1;
-- 文房具の在庫を補充
UPDATE product_stocks
SET quantity = quantity + 20
WHERE stationery_id = 2;
-- ここで何らかのエラーが発生したと仮定
ROLLBACK; -- エラーが発生したのでロールバック
```

---

## ✅ 振り返りチェック

- [ ] UPDATE文の基本構文を理解した
- [ ] DELETE文の基本構文を理解した
- [ ] トランザクションの基本を理解した
- [ ] トランザクションを使ったデータ更新の実行方法を理解した
- [ ] トランザクションのロールバックを実行できる
- [ ] 次の内容へつながる疑問が出てきた？

---
