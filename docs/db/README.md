# データベースの使い方について

今回は書籍管理するために、データベースを利用します。  
現代のWebアプリケーションは、情報(データ)保持するために必ずと言っていいほどリレーショナルデータベースを使います。

今回はデータベース製品としてMySQLを使います。


## そもそもデータベースとはなにか？


https://blog.codecamp.jp/database-basic-beginner


## SQLについて

データベースを操作するために、SQLという言語を利用します。
基本的には SELECT, INSERT, UPDATE, DELETEの４つの構文を使います。


たとえばデータを取得する場合はこのように操作します。

```SQL
SELECT * FROM books
```

データを登録するときは以下のようにします。


```SQL
INSERT INTO books (isbn, name, image) VALUES ('978-4-08-882495-6', '鬼滅の刃 23', 'https://cover.openbd.jp/9784088824956.jpg')
```

# 実際にデータベースを操作してみよう

実際にデータベースを触ってみましょう。
ここでは以下の作業をします。

- テーブルの作成
- データをテーブルに登録する
- 登録したデータを表示する


## テーブルの作成


まずはDBに接続します。


```shell
$ docker compose exec db /bin/bash
$ mysql -u root -proot booklist
```

まずはテーブルがあるか確かめましょう。
確かめるためには `show table` コマンドを実行します。

```shell
mysql> show tables;
Empty set (0.00 sec)
```

この時点ではテーブルがありません。
まずはテーブルを作成しましょう。

```sql
create table books (
  id INT AUTO_INCREMENT NOT NULL, 
  name VARCHAR(255) NOT NULL, 
  isbn VARCHAR(255) NOT NULL, 
  image VARCHAR(255), 
  readed TINYINT(1) NOT NULL DEFAULT 0,
  created_at DATETIME NOT NULL, 
  updated_at DATETIME NOT NULL,
  PRIMARY KEY(id)
) DEFAULT CHARACTER SET utf8mb4 COLLATE `utf8mb4_unicode_ci` ENGINE = InnoDB;
```

もう一度 `show tables` をすると booksテーブルが作成さています。

### テーブルの削除

テーブルを消したい場合はdrop文を使います。

```
drop table テーブル名
```

で消すことができます。


## データをテーブルに登録する

テーブルを作成したら、次にデータを入れてみましょう。

```sql
INSERT INTO books (isbn, name, image, created_at, updated_at) VALUES ('978-4-08-882495-6', "鬼滅の刃 23", 'https://cover.openbd.jp/9784088824956.jpg', now(), now())
```

## 登録したデータを表示する

データを登録したら、次はデータを確認してみましょう。


```sql
SELECT * FROM books;
```

これでbooksに登録されているすべてのデータを確認できます。  
また条件を絞ることもできます。  
たとえばISBNで絞る場合は以下のようにします。  

```sql
SELECT * FROM books WHERE isbn = '978-4-08-882495-6';
```







