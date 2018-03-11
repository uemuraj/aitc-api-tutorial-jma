# 気象庁防災情報 XML API チュートリアル

気象庁防災情報XMLをご存知でしょうか。天気予報や警報を主に、多種多様な情報が気象庁から公開されています。  
現在、ユーザ登録をすれば誰でも無料で受け取ることができます。

* [気象庁防災情報XMLフォーマット形式電文の公開（PUSH型）について](http://xml.kishou.go.jp/open_trial/index.html)

実際、どんな情報が来るのか簡単に見てみましょう。

* [気象庁防災情報 XML 一覧](http://api.aitc.jp/jmardb/)

このチュートリアルでは、上記のサイトのAPIを利用して、ある地域の天気予報を検索してみます。


## 検索方法

[気象庁防災情報 XML 検索 API 仕様](http://api.aitc.jp/jmardb-api/help) にあるとおりなのですが、順を追って試して行きましょう。


### トップページにあるフォームを使う

まずは [気象庁防災情報 XML 一覧](http://api.aitc.jp/jmardb/) に検索フォームがあるので、これを試しましょう。

* 「タイトル検索(1)」で **府県天気** と入力して「検索」ボタンを押す

府県天気予報、府県天気概況といった情報が一覧に表示されるはずです。  
リンク先は気象庁から公開された「生」のデータを保存したものです。  
リンク先にジャンプすると、ブラウザでそのまま内容を見ることができます。  
「生」のデータを見た時に１つ目の `<Title>` に書かれている内容を検索するのが「タイトル検索(1)」です。

ある地域の天気予報に絞るには、次の手順が簡単です。

* 「タイトル検索(2)」で **岩手県府県天気予報** のように入力して「検索」ボタンを押す

「タイトル検索(2)」は完全一致で検索するので、入力が正しくないと何も見つかりません。  
「生」のデータを見た時に２つ目の `<Title>` に書かれているものを入力してください。

もっと地域を絞り込むには「地区コード」を使います。

* トップページの一番下にある「地区コード検索（気象）」で `030010` と入力して「検索」ボタンを押す

`030010` というのは、「生」のデータを見た時に `<Area>` の中の `<Code>` にある数値です。  
この検索では「岩手県の内陸」を表す地区コードで検索したので、天気予報だけでなく警報・注意報なども表示されたと思います。  
警報・注意報は `<Area>` がたくさん含まれていて「○○市」「○○地域」といったものがあり、それぞれ地区コードが付いているのが分るでしょう。

> 地区コードの一覧表も公開されています。  
> 気象庁の [気象庁防災情報XMLフォーマット 技術資料](http://xml.kishou.go.jp/tec_material.html) に「個別コード表」というものがあります。  
> 圧縮されていますが、ダウンロードして展開すると Excel ファイルが複数入っています。  
> この中の ` 20161013_AreaInformationCity-AreaForecastLocalM.xls` を見てみてください。


### REST API を使う

次に [REST API](http://api.aitc.jp/jmardb-api/help) で検索してみましょう。  
ブラウザで簡単に試すことができますが、Linux や Mac では `curl` コマンドを使うのも良いと思います。

以下の URL にアクセスしてみてください。結果が返ってくるまでかなり時間がかかるので注意してください。  
URL の中の `order=new` は新しいものから順に検索するという意味で、 `%E5%BA%9C%E7%9C...` というのは「府県天気概況」を URL エンコードしたものです。

* http://api.aitc.jp/jmardb-api/search?order=new&title=%E5%BA%9C%E7%9C%8C%E5%A4%A9%E6%B0%97%E6%A6%82%E6%B3%81

検索期間を指定すると速くなります。 `datetime` を **２つ** 書いてみてください。  
以下の URL は「2018年1月1日以上」「2018年1月4日未満」の「府県天気概況」を検索できます。

* http://api.aitc.jp/jmardb-api/search?datetime=2018-01-01&datetime=2018-01-04&title=%E5%BA%9C%E7%9C%8C%E5%A4%A9%E6%B0%97%E6%A6%82%E6%B3%81

> `curl` コマンドを使う場合：
> URL をダブルクォーテーションで囲んでください。
> ```
> curl "http://api.aitc.jp/jmardb-api/search?datetime=2018-01-01&datetime=2018-01-04&title=%E5%BA%9C%E7%9C%8C%E5%A4%A9%E6%B0%97%E6%A6%82%E6%B3%81"
> ```

アクセスした結果どちらも XML でなく JSON のデータが表示されたと思います。  
これは「生」のデータではなく、検索結果だけを表したデータです。  
この中で `link` に書かれている URL が検索フォームで検索したの時の「生」のデータの URL と同じものです。

```
"link":"http://api.aitc.jp/jmardb-api/reports/d60a1719-42ad-37d6-9b4e-a63d8d5dde8a"
```

「生」のデータを XML でなく JSON で見ることもできます。最後に `.json` を付けた以下の URL にアクセスしてみてください。

* http://api.aitc.jp/jmardb-api/reports/d60a1719-42ad-37d6-9b4e-a63d8d5dde8a.json

JSON の一部だけが欲しい時は、以下のようにすることができます。

* http://api.aitc.jp/jmardb-api/reports/d60a1719-42ad-37d6-9b4e-a63d8d5dde8a.json?path=/report/body/comment

検索と同時に、JSON の一部だけを抜き出すことができます。以下の URL にアクセスして JSON の中の `fragment` の内容を見てください。

* http://api.aitc.jp/jmardb-api/search?datetime=2018-01-01&datetime=2018-01-04&title=%E5%BA%9C%E7%9C%8C%E5%A4%A9%E6%B0%97%E6%A6%82%E6%B3%81&path=/report/body/comment


### SPARQL を使う

さらに、天気予報以外のものも検索してみましょう。 [SPARQL クエリ発行](http://api.aitc.jp/ds/) を試しましょう。  
天気予報以外にどんな情報があるのかは [気象庁防災情報XMLフォーマット 技術資料](http://xml.kishou.go.jp/tec_material.html) で公開されていますが、実際に来ている情報から検索できます。

まずは、ブラウザで http://api.aitc.jp/ds/ にアクセスしてください。

```
PREFIX xsd:   <http://www.w3.org/2001/XMLSchema#>
PREFIX atom:  <http://www.w3.org/2005/Atom#>
PREFIX jma:   <http://cloud.projectla.jp/jma/>
PREFIX area:  <http://cloud.projectla.jp/jma/area#>
PREFIX link2: <http://cloud.projectla.jp/jma/link2/>

## Tripleを取得するサンプル
# SELECT * WHERE {?s ?p ?o} LIMIT 100
## 述語のリスト
# SELECT DISTINCT ?p WHERE {?s ?p ?o} ORDER BY ?p
## エリア名とコードのリスト
# SELECT DISTINCT ?name ?code WHERE {?o area:name ?name . ?o area:code ?code } ORDER BY ?code
## タイトルで集計
SELECT ?title (COUNT(?id) as ?c) WHERE {?id atom:title ?title} GROUP BY ?title ORDER BY DESC(?c)
## 日付で集計
# SELECT ?date (COUNT(?date) as ?c) WHERE { SELECT ?id (SUBSTR(?updated,1,10) as ?date) WHERE {?id atom:updated ?updated } } GROUP BY ?date ORDER BY ?date
## 特定のタイトルのものの中身を参照
# SELECT ?updated ?content ?link WHERE {?id atom:title "火山の状況に関する解説情報" . ?id atom:updated ?updated . ?id atom:content ?content . ?id atom:link ?link} ORDER BY ?updated
## 「雪」に関するコンテンツがどの気象台からいくつ出ているか？
# SELECT ?author (COUNT(?id) as ?c) WHERE {?id atom:author ?author . ?id atom:content ?content . FILTER(REGEX(?content, "雪")) } GROUP BY ?author ORDER BY DESC(?c)
## "全般週間天気予報"のうち、2014/01/01に配信されたもの
# SELECT ?id ?updated ?content ?link WHERE {?id atom:title "全般週間天気予報" . ?id atom:updated ?updated . ?id atom:content ?content . ?id atom:link ?link . FILTER (xsd:dateTime(?updated) >= "2013-01-01T00:00:00+09:00"^^xsd:dateTime && xsd:dateTime(?updated) < "2013-01-02T00:00:00+09:00"^^xsd:dateTime)} ORDER BY ?updated
```

上記の `SELECT` から始まるのが *SPARQL* という問い合わせ言語の文です。 *SQL* に似ていますが少し違います。  
行の最初にある `#` はコメントという意味です。 `#` が２つある行は SPARQL 文の説明になっています。  
以下の部分だけ `#` が付いていないのに気づきましたか？このまま「Get Result」ボタンを押すと、この問い合わせが実行されます。

```
## タイトルで集計
SELECT ?title (COUNT(?id) as ?c) WHERE {?id atom:title ?title} GROUP BY ?title ORDER BY DESC(?c)
``` 

この問い合わせの結果は、これまでに受け取った情報のタイトルとその件数になっています。  
タイトルは [気象庁防災情報 XML 一覧](http://api.aitc.jp/jmardb/) に出ているものと同じです。

「火山現象に関する海上警報・海上予報」というタイトルの情報が 12 件だけありますね。  
これを REST API で検索してみましょう。

* http://api.aitc.jp/jmardb-api/search?order=new&title=%E7%81%AB%E5%B1%B1%E7%8F%BE%E8%B1%A1%E3%81%AB%E9%96%A2%E3%81%99%E3%82%8B%E6%B5%B7%E4%B8%8A%E8%AD%A6%E5%A0%B1%E3%83%BB%E6%B5%B7%E4%B8%8A%E4%BA%88%E5%A0%B1

API の検索結果の中から「生」のデータを１つ選んで見てみましょう。

* http://api.aitc.jp/jmardb-api/reports/c0895449-9575-3218-8028-f7b6cd37ba2c

火山に関する情報は VolcanoInfoContent にあるものが読み易そうです。検索と同時に、この部分だけ抜き出してみましょう。

XML としては http://xml.kishou.go.jp/jmaxml1/body/volcanology1/ という名前区間の中の VolcanoInfoContent 要素ですが、  
REST API では XML を JSON に加工してから抜き出すので、簡単に書くことができます。

* http://api.aitc.jp/jmardb-api/search?order=new&path=/report/body/volcanoInfoContent&title=%E7%81%AB%E5%B1%B1%E7%8F%BE%E8%B1%A1%E3%81%AB%E9%96%A2%E3%81%99%E3%82%8B%E6%B5%B7%E4%B8%8A%E8%AD%A6%E5%A0%B1%E3%83%BB%E6%B5%B7%E4%B8%8A%E4%BA%88%E5%A0%B1
