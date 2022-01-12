# Web APIの使い方について

Web APIがなにかわからない場合は、こちらの記事を参照してください。
[Web APIとは何なのか](https://qiita.com/NagaokaKenichi/items/df4c8455ab527aeacf02)

## 今回使う書籍検索のWeb APIについて

今回は[openbd](https://openbd.jp) を通じてWeb APIの使い方を学びます。

書籍にはかならずISBNというユニークなコードが割り振られます。
このISBNを検索して書籍情報を取得します。


phpではいくつか外部のAPIにリクエストを送る方法がありますが、今回は `file_get_contents` 関数を使ってみましょう。

```php
<?php

// book1.php

const API_URL ='https://api.openbd.jp/v1/get?isbn=%s'; // コールするAPIのベースとなるURLを定数で定義

$isbn = '978-4-08-882495-6'; // 検索する本のISBN

$url = sprintf(API_URL, $isbn); // 検索するURLを生成 https://api.openbd.jp/v1/get?isbn=978-4-08-882495-6

$content = file_get_contents($url); // APIからデータを取得する

$json = json_decode($content, true); // JSON形式になっているので、PHPの配列に変換

echo "<pre>";
print_r($json);
?>
```

これでデータが取得できました。

## HTMLをつかって取得した情報をブラウザで表示する

print_rでの出力だと画像が表示できないので、HTMLを使って画像などの情報を表示させてみます。


```php
<?php

// book2.php

const API_URL ='https://api.openbd.jp/v1/get?isbn=%s'; // コールするAPIのベースとなるURL

$isbn = '978-4-08-882495-6'; // 検索する本のISBN

$url = sprintf(API_URL, $isbn); // 検索するURLを生成 https://api.openbd.jp/v1/get?isbn=978-4-08-882495-6

$content = file_get_contents($url); // APIからデータを取得する

$json = json_decode($content, true); // JSON形式のデータをPHPの配列に変換

?>
<html lang='ja'>
  <head>
    <title>書籍管理システム</title>
  </head>
  <body>
    <?php foreach($json as $data) : ?>
    <div>
      <h2><?php echo $data['summary']['title'] ?></h2>
      <p>
        <?php echo $data['summary']['pubdate'] ?>
      </p>
      <p>
        <img src="<?php echo $data['summary']['cover'] ?>" />
      </p>
    </div>
    <?php endforeach ?>
  </body>
</html>
```

上記のコードをブラウザで実行すると以下のように表示されます。

![](/images/img1.png)

<p class="alert">
ちなみに最近のPHPはこのようにHTMLファイルの中にPHPのコードの埋め込むようなことはセキュリティ的な観点からあまり推奨されません。  <br>
テンプレートエンジンやフレームワークを利用して安全にHTMLとして出力する方法を利用すること強くおすすめします。
</p>


## ISBNを動的に変更できるようにする

鬼滅の刃以外の書籍を検索するときにどうしたらいいでしょうか？
毎回ソースコードを変更するわけにもいきません。


phpでは`https://localhost?isbn=xxxxx` という形式で渡せば、ISBNを動的に設定できますよね。
こういった方法はよく使われます。

例えばGoogleの場合は[https://www.google.com/search?q=%E9%9D%99%E5%B2%A1](https://www.google.com/search?q=%E9%9D%99%E5%B2%A1) といった `q=xxxx` というクエリを渡しています。


今回 `isbn=xxxxx` というクエリを受け取ってISBNを設定できるようにしましょう。

```php
$isbn = '978-4-08-882495-6'; // 検索する本のISBN
```

上記を以下のように変更してください。


```php
$isbn = $_GET['isbn']
```

あとは [http://localhost/book.php?isbn=978-4088770796](http://localhost/book.php?isbn=978-4088770796) アクセスすれば違う漫画の情報が表示されます。

ISBNを調べて好きな書籍を表示させてみてください。



