# Day2_SELECT文の基本

---

## 🎯 学習目標
- SELECT文でテーブルから一覧を取得する
- WHERE句を使用して条件を指定してデータを取得する
- JOIN句を使用して複数のテーブルを結合してデータを取得する
- GROUP BY句を使用してデータを集計する
- HAVING句を使用して集計結果に条件を指定する
---

## 📘 内容概要

- SELECT文の基本構文と使用方法を学ぶ
- WHERE句を使用して条件を指定する方法を学ぶ
- JOIN句を使用して複数のテーブルを結合する方法を学ぶ
- GROUP BY句を使用してデータを集計する方法を学ぶ
- HAVING句を使用して集計結果に条件を指定する方法を学ぶ
---

## 📝 内容詳細

1. まずはretister_usersテーブルから全てのレコードを取得する簡単なSQLを書いて実行してみましょう。
   ```sql
   SELECT * FROM register_users;
   ```
   上記SQLをSMSSのクエリエディタに入力し、実行してみてください。
   - `SELECT *` は全ての列を選択することを意味します。
   - `FROM register_users` は `register_users` テーブルからデータを取得することを意味します。
   - 実行結果として、`register_users` テーブルの全てのレコードが表示されるはずです。

2. 次に、`register_users` テーブルから特定の列だけを取得するSQLを書いてみましょう。
   ```sql
   SELECT id, name FROM register_users;
   ```
   - 上記SQLは、`id` と `name` 列だけを取得することを意味します。
   - 実行結果として、`id` と `name` 列のデータが表示されるはずです。
3. WHERE句を使用して、特定の条件に一致するレコードを取得するSQLを書いてみましょう。
   ```sql
   SELECT * FROM register_users WHERE name = '田中 太郎';
;
   ```
   - 上記SQLは、`name` 列が '田中 太郎' のレコードだけを取得することを意味します。
   - 実行結果として、`name` が '田中 太郎' のレコードが表示されるはずです。
4. JOIN句を使用して、複数のテーブルを結合してデータを取得するSQLを書いてみましょう。
   ```sql
      SELECT
         rs.id AS sale_id,
         ru.name AS register_user,
         COALESCE(b.title, s.name) AS product_name,
         rs.sale_date,
         rs.sale_amount
      FROM register_sales rs
      JOIN register_users ru ON rs.register_user_id = ru.id
      JOIN product_stocks ps ON rs.product_stock_id = ps.id
      LEFT JOIN books b ON ps.book_id = b.id
      LEFT JOIN stationery s ON ps.stationery_id = s.id
      ORDER BY rs.sale_date;
   ```
   - 上記SQLは、`register_sales` テーブルと `register_users` テーブルを結合して、売上の詳細を取得することを意味します。
   - `JOIN` 句は、2つのテーブルを結合するために使用されます。
   - `ON` 句は、結合条件を指定するために使用されます。
   - `LEFT JOIN` は、左側のテーブルの全てのレコードを取得し、右側のテーブルに一致するレコードがない場合はNULLを返すことを意味します。
   - `COALESCE` 関数は、複数の引数の中から最初に非NULLの値を返す関数です。
   - 実行結果として、売上の詳細が表示されるはずです。
   - `ORDER BY rs.sale_date` は、売上日付で結果をソートすることを意味します。
   - `AS` は、列のエイリアスを指定するために使用されます。ここでは、`rs.id` を `sale_id` として、`ru.name` を `register_user` として、`COALESCE(b.title, s.name)` を `product_name` として表示しています。
   - `COALESCE(b.title, s.name)` は、書籍のタイトルが存在する場合はそれを、存在しない場合は文房具の名前を表示することを意味します。
   - 実行結果として、売上の詳細が表示されるはずです。
   - もし、`register_sales` テーブルにデータがない場合は、結果が空になることがあります。
   - また、`register_users` テーブルにデータがない場合も、結果が空になることがあります。
   - `product_stocks` テーブルにデータがない場合は、売上の詳細が表示されないことがあります。
   - `books` テーブルにデータがない場合は、書籍のタイトルがNULLになることがあります。
   - `stationery` テーブルにデータがない場合は、文房具の名前がNULLになることがあります。
   - 実行結果として、売上の詳細が表示されるはずです。
   - もし、結合条件が正しくない場合は、結果が空になることがあります。
   - また、結合条件が正しい場合でも、結合するテーブルにデータがない場合は、結果が空になることがあります。
   - 例えば、`register_users` テーブルはマスターデータと呼ばれるテーブルの一つで、ユーザー情報を管理しています。ここにデータを入力するにはマスター管理ツールのようなアプリケーション上の仕組みが必要となります。ここではマスター管理ツールを用いずダミーデータを用意することでデータの表示を可能としています。

5. 次は GROUP BY句を使用して、商品ごとの売り上げを表示していましょう
   ```sql
   SELECT
      COALESCE(b.title, s.name) AS product_name,
      SUM(rs.sale_amount) AS total_sales,
      COUNT(rs.id) AS sale_count
   FROM register_sales rs
   JOIN product_stocks ps ON rs.product_stock_id = ps.id
   LEFT JOIN books b ON ps.book_id = b.id
   LEFT JOIN stationery s ON ps.stationery_id = s.id
   GROUP BY COALESCE(b.title, s.name)
   ORDER BY total_sales DESC;
   ```
   - 上記SQLは、売上の詳細を商品ごとに集計し、商品名、売上合計、売上件数を表示することを意味します。
   - `GROUP BY` 句は、指定した列でグループ化して集計を行うために使用されます。
   - `SUM(rs.sale_amount)` は、売上金額の合計を計算するために使用されます。
   - `COUNT(rs.id)` は、売上件数をカウントするために使用されます。
   - `COALESCE(b.title, s.name)` は、書籍のタイトルが存在する場合はそれを、存在しない場合は文房具の名前を表示することを意味します。
   - `ORDER BY total_sales DESC` は、売上合計で結果を降順にソートすることを意味します。
   - 実行結果として、商品ごとの売上合計と売上件数が表示されるはずです。

6. 次はレジ操作者ごとの売上合計を出してみましょう
``` sql
   SELECT
      ru.name AS register_user,
      COUNT(rs.id) AS sale_count,
      SUM(rs.sale_amount) AS total_sales
   FROM register_sales rs
   JOIN register_users ru ON rs.register_user_id = ru.id
   GROUP BY ru.name
   ORDER BY total_sales DESC;
```
   - 上記SQLは、レジ操作者ごとの売上合計と売上件数を表示することを意味します。
   - `ru.name` はレジ操作者の名前を表示するために使用されます。
   - `COUNT(rs.id)` は、レジ操作者ごとの売上件数をカウントするために使用されます。
   - `SUM(rs.sale_amount)` は、レジ操作者ごとの売上金額の合計を計算するために使用されます。
   - `GROUP BY ru.name` は、レジ操作者ごとにグループ化して集計を行うために使用されます。
   - `ORDER BY total_sales DESC` は、売上合計で結果を降順にソートすることを意味します。
   - 実行結果として、レジ操作者ごとの売上合計と売上件数が表示されるはずです。

7. 日別の売上合計を出してみましょう
``` sql
   SELECT
      CAST(rs.sale_date AS DATE) AS sale_day,
      COUNT(*) AS sale_count,
      SUM(rs.sale_amount) AS total_sales
   FROM register_sales rs
   GROUP BY CAST(rs.sale_date AS DATE)
   ORDER BY sale_day;
```
   - 上記SQLは、日別の売上合計と売上件数を表示することを意味します。
   - `CAST(rs.sale_date AS DATE)` は、売上日付を日単位に変換して表示するために使用されます。
   - `COUNT(*)` は、日別の売上件数をカウントするために使用されます。
   - `SUM(rs.sale_amount)` は、日別の売上金額の合計を計算するために使用されます。
   - `GROUP BY CAST(rs.sale_date AS DATE)` は、日別にグループ化して集計を行うために使用されます。
   - `ORDER BY sale_day` は、売上日付で結果を昇順にソートすることを意味します。
   - 実行結果として、日別の売上合計と売上件数が表示されるはずです。

8. 商品メーカー別の販売件数
   ``` sql
   SELECT
      COALESCE(bm.name, sm.name) AS maker_name,
      COUNT(rs.id) AS sale_count,
      SUM(rs.sale_amount) AS total_sales
   FROM register_sales rs
   JOIN product_stocks ps ON rs.product_stock_id = ps.id
   LEFT JOIN books b ON ps.book_id = b.id
   LEFT JOIN book_makers bm ON b.book_maker_id = bm.id
   LEFT JOIN stationery s ON ps.stationery_id = s.id
   LEFT JOIN stationery_makers sm ON s.stationery_maker_id = sm.id
   GROUP BY COALESCE(bm.name, sm.name)
   ORDER BY total_sales DESC;
   ```
   - 上記SQLは、商品メーカー別の販売件数と売上合計を表示することを意味します。
   - `COALESCE(bm.name, sm.name)` は、書籍メーカーの名前が存在する場合はそれを、存在しない場合は文房具メーカーの名前を表示することを意味します。
   - `COUNT(rs.id)` は、商品メーカー別の販売件数をカウントするために使用されます。
   - `SUM(rs.sale_amount)` は、商品メーカー別の売上金額の合計を計算するために使用されます。
   - `GROUP BY COALESCE(bm.name, sm.name)` は、商品メーカー別にグループ化して集計を行うために使用されます。
   - `ORDER BY total_sales DESC` は、売上合計で結果を降順にソートすることを意味します。
   - 実行結果として、商品メーカー別の販売件数と売上合計が表示されるはずです。

9. 補足 GROUP BY 句を使うカラムは
   - SELECT に出す列のうち、集計関数を使っていない列はすべて GROUP BY に書く必要がある
   - 逆に、集計関数（SUM, COUNT, MAX など）を使う列は GROUP BY に書かなくてもよい

10. HAVING句の説明をします。
      - `HAVING` 句は、`GROUP BY` 句でグループ化された結果に対して条件を指定するために使用されます。
      - `HAVING` 句は、集計関数を使用した条件を指定することができます。
      - `WHERE` 句は、グループ化前のデータに対して条件を指定するために使用されますが、`HAVING` 句はグループ化後のデータに対して条件を指定するために使用されます。
11. 売上合計が1000以上のレコードだけを表示する場合は、以下のように書きます。
      ```sql
      SELECT
         COALESCE(b.title, s.name) AS product_name,
         SUM(rs.sale_amount) AS total_sales,
         COUNT(rs.id) AS sale_count
      FROM register_sales rs
      JOIN product_stocks ps ON rs.product_stock_id = ps.id
      LEFT JOIN books b ON ps.book_id = b.id
      LEFT JOIN stationery s ON ps.stationery_id = s.id
      GROUP BY COALESCE(b.title, s.name)
      HAVING SUM(rs.sale_amount) >= 1000
      ORDER BY total_sales DESC;
      ```
      - 上記SQLは、売上合計が1000以上のレコードだけを表示することを意味します。
      - `HAVING` 句は、`GROUP BY` 句でグループ化された結果に対して条件を指定するために使用されます。
      - `HAVING SUM(rs.sale_amount) >= 1000` は、売上合計が1000以上のレコードだけを表示することを意味します。
      - 実行結果として、売上合計が1000以上のレコードが表示されるはずです。
      - `HAVING` 句は、`GROUP BY` 句の後に記述する必要があります。
      - `HAVING` 句は、`GROUP BY` 句でグループ化された結果に対して条件を指定するために使用されます。
      - `HAVING` 句は、集計関数を使用した条件を指定することができます。
      - `HAVING` 句は、`WHERE` 句と同様に、条件を指定するために使用されますが、`WHERE` 句はグループ化前のデータに対して条件を指定するために使用されます。
      - `HAVING` 句は、`GROUP BY` 句でグループ化された結果に対して条件を指定するために使用されます。
      - `HAVING` 句は、集計関数を使用した条件を指定することができます。
      - `HAVING` 句は、`GROUP BY` 句の後に記述する必要があります。   
12. 売上回数が1回以上のレジ操作者
```sql
   SELECT
      ru.name AS register_user,
      COUNT(rs.id) AS sale_count,
      SUM(rs.sale_amount) AS total_sales
   FROM register_sales rs
   JOIN register_users ru ON rs.register_user_id = ru.id
   GROUP BY ru.name
   HAVING COUNT(rs.id) >= 1
   ORDER BY total_sales DESC;
```
   - 上記SQLは、売上回数が1回以上のレジ操作者の名前、売上件数、売上合計を表示することを意味します。
   - `HAVING COUNT(rs.id) >= 1` は、売上回数が1回以上のレジ操作者だけを表示することを意味します。
   - 実行結果として、売上回数が1回以上のレジ操作者の名前、売上件数、売上合計が表示されるはずです。

13. 1日に1件以上売れた日の売上合計を表示
```sql
   SELECT
      CAST(rs.sale_date AS DATE) AS sale_day,
      COUNT(*) AS sale_count,
      SUM(rs.sale_amount) AS total_sales
   FROM register_sales rs
   GROUP BY CAST(rs.sale_date AS DATE)
   HAVING COUNT(*) >= 1
   ORDER BY sale_day;
```
   - 上記SQLは、1日に1件以上売れた日の売上合計と売上件数を表示することを意味します。
   - `HAVING COUNT(*) >= 1` は、1日に1件以上売れた日のみを表示することを意味します。
   - 実行結果として、1日に1件以上売れた日の売上合計と売上件数が表示されるはずです。 

14. まとめ：WHERE と HAVING の違い
- `WHERE` 句は、グループ化前のデータに対して条件を指定するために使用されます。
- `HAVING` 句は、グループ化後のデータに対して条件を指定するために使用されます。
- `WHERE` 句は、集計関数を使用した条件を指定することができません。
- `HAVING` 句は、集計関数を使用した条件を指定することができます。

---

## ✅ 振り返りチェック

- [ ] SELECT文の基本構文を理解した
- [ ] WHERE句を使用して条件を指定してデータを取得できるようになった
- [ ] JOIN句を使用して複数のテーブルを結合してデータを取得できるようになった
- [ ] GROUP BY句を使用してデータを集計できるようになった
- [ ] HAVING句を使用して集計結果に条件を指定できるようになった

---
