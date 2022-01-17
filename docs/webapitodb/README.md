# Web APIから取得した情報をデータベースに登録する

次はWebAPIとデータベースを組み合わせて、ISBNを入力したら登録出来る機能を作ってみましょう。


## 書籍の登録

実装のサンプルはこのような形です。

```php
<?php

$action = $_GET['action'] ?? null;
$isbn = $_POST['isbn'] ?? null;

/**
 * 書籍が登録されているか判定する
 */
$hasBook = function(PDO $pdo, string $isbn): bool {
  $isbn = str_replace('-', '',$isbn);

  // すでに登録があるか確認する
  $stmt = $pdo->prepare("SELECT * FROM books WHERE isbn = :isbn"); // クエリの実行
  $stmt->bindParam(':isbn', $isbn);
  $stmt->execute();

  return !empty($stmt->fetchAll());
};


/**
 * 検索用の関数
 */
$search = function(?string $isbn):array {
  $url = sprintf('https://api.openbd.jp/v1/get?isbn=%s', $isbn); // 検索するURLを生成 https://api.openbd.jp/v1/get?isbn=978-4-08-882495-6
  $content = file_get_contents($url); // APIからデータを取得する
  $json = json_decode($content, true)[0]; // JSON形式になっているので、PHPの配列に変換

  return [true, $json['summary']] ?? [false, []];
};

/**
 * 登録用関数
 */
$register = function(?string $isbn) use ($search, $hasBook):array {

  if ($isbn === null) {
    // ISBNがない
    return [false, []]; 
  }


  $book = $search($isbn)[1];

  if (empty($book)) {
    // ISBNから本が見つからない
    return [false, []];
  }

  $pdo = new PDO('mysql:host=db;port=3306;dbname=booklist', 'root', 'root'); // データベースへの接続
  if ($hasBook($pdo, $isbn)) {
    // すでにデータベースに登録されているのでここで処理を抜ける
    return [false, $book];
  }

  $stmt = $pdo->prepare("INSERT INTO books (isbn, name, image, created_at, updated_at) VALUES (:isbn, :name, :image, now(), now())");
  $stmt->bindValue(':isbn', $book['isbn']);
  $stmt->bindValue(':name', $book['title']);
  $stmt->bindValue(':image', $book['cover']);
  $stmt->execute();

  return [true, $book];
};


[$state, $book] = match ($action) {
  'search' => $search($isbn),
  'register' => $register($isbn),
  null => [false, []],
};
?>
<html>
  <head>
    <title>書籍管理システム</title>
  </head>
  <body>
    <div>
      <?php if ($action === 'register' && $state === true): ?>
        <p style="color: #red">登録しました</p>
      <?php endif; ?>
    </div>
    <div>
      <h3>検索する</h3>
      <div>
        <form method="post" action="?action=search">
          <input type="text" name="isbn" value="<?php echo $isbn ?>" />
          <button type="submit">探す</button>
        </form>
      </div>
    </div>


    <?php if (!empty($book)) : ?>
    <div>
      <h2><?php echo $book['title'] ?></h2>
      <p>
        <?php echo $book['pubdate'] ?>
      </p>
      <p>
        <img src="<?php echo $book['cover'] ?>" />
      </p>
      <form method="post" action="?action=register">
        <input type="hidden" name="isbn" value="<?php echo $isbn ?>">
        <button type="submit">書籍を登録する</button>
      </form>
    </div>
    <?php endif; ?>
  </body>
</html>
```