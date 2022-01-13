# phpからデータベースを操作する

まずは先程登録したデータをブラウザで表示してみましょう。


```php
<?php
// db.php

$pdo = new PDO('mysql:host=db;port=3306;dbname=booklist', 'root', 'root'); // データベースへの接続
$stmt = $pdo->query("SELECT * FROM books"); // クエリの実行

?>
<html>
  <body>
    <table>
      <thead>
        <tr>
          <th>ID</th>
          <th>書籍名</th>
          <th>画像</th>
          <th>既読/未読</th>
          <th>登録日</th>
          <th>更新日</th>
          <th></th>
        </tr>
      </thead>
      <?php foreach ($stmt as $books) : ?>
      <tr>
        <td><?php echo $books['id'] ?></td>
        <td><?php echo $books['name'] ?></td>
        <td>
          <?php if ($books['image'] !== null) : ?>
            <img src="<?php echo $books['image'] ?>" />
          <?php endif; ?>
        </td>
        <td>
            <?php echo $books['readed'] === 1 ? '既読' : '未読' ?>
        </td>
        <td>
          <?php echo (new \DateTime($books['created_at']))->format('Y年m月d日 H時i分s秒') ?>
        </td>
        <td>
          <?php echo (new \DateTime($books['updated_at']))->format('Y年m月d日 H時i分s秒') ?>
        </td>
        <td></td>
      </tr>
      <?php endforeach; ?>
    </table>
  </body>
</html>
```

## データベースの情報を更新してみる

URLのクエリパラメータを見て更新するかの判断をします。

```
http://localhost/db.php?action=update&id=1
```

このように
actionというパラメータに `update`, idには更新したいIDをパラメータにセットすると更新できるという仕様にしてみます。

```php
<?php
$pdo = new PDO('mysql:host=db;port=3306;dbname=booklist', 'root', 'root'); // データベースへの接続

if (isset($_GET['action']) && $_GET['action'] === 'update' && isset($_GET['id'])) {
  // 既読処理を行う
  $stmt = $pdo->prepare('UPDATE books SET readed = 1 WHERE id = ?');
  $stmt->bindValue(1, $_GET['id'],PDO::PARAM_INT);
  $stmt->execute();
}


$stmt = $pdo->query("SELECT * FROM books"); // クエリの実行

?>
```

さらに更新用のリンクをHTMLのテーブルタグのなかに用意します用意します。

```php
        <td>
          <?php if ($books['readed'] === 0): ?>
            <a href="<?php echo sprintf('?action=update&id=%d', $books['id']); ?>">既読にする</a>
          <?php endif; ?>
        </td>
```
