# Monacaとニフティクラウドmobile backendでWebコンテンツのスマホアプリ化を体験してみよう
MonacaのRSSリーダーのサンプルをベースに、ニフティクラウドmobile backendを使ったお気に入り機能を実装したRSSリーダー（JavaScript SDK version 2 対応）

* [fav-rss-reader-v2](https://goo.gl/KQRcQy)として公開していましたサンプルプロジェクトをハンズオンセミナー向けに一部改良し掲載しています。(2016/1/26)

## 使い方

### お気に入り機能付きRSSリーダー＜オフライン版＞

* このリポジトリをそのままMonacaへインポートすればお試し頂けます。
* 端末のローカルストレージにお気に入り情報を保存します。
* 以下の手順に従ってインポート・動作確認をしてください。

1. [Download ZIP](https://github.com/ndyuya/fav-rss-reader-v2/archive/master.zip)を右クリックしてURLをコピーする
1. [Monaca](https://ja.monaca.io/)で「新規プロジェクト」＞「Import Project」＞インポート方法のURLを指定してインポートに貼り付ける＞「インポート」をクリック
1. プロジェクトを「開く」をクリックして開いたら、[Monacaデバッガー](http://ja.monaca.io/debugger.html)で動作確認をする

### お気に入り機能付きRSSリーダー＜オンライン版＞

* オフライン版に手を加えることでオンライン版を作ることができます。
* mobile backendのデータストアへお気に入り情報を保存します。
* 以下の手順に従ってオフライン版を改変してください。(予めオフライン版の動作確認まで完了しておく必要があります。)

1. JavaScript SDKを入手する
 *  「設定」＞「JS/CSSコンポーネントの追加と削除…」を選択
 * 右上の検索欄に「ncmb」と入力して検索ボタンをクリックする
 * 「追加」をクリック
 * バージョン(2.0.2)はそのままで、「インストール開始」をクリック
 * ｢components/ncmb/min.js｣にチェックをつけて｢OK｣を選択

1. index.htmlの12行を以下のように書き換える
 * 変更前
    <script src="js/favorite-offline.js"></script>
 * 変更後
    <script src="js/favorite-online.js"></script>

1. [mobile backendのダッシュボード](https://console.mb.cloud.nifty.com/)で新しいアプリを作成する
1. 作成したアプリのアプリケーションキーをコピーして、Monaca IDE上のindex.htmlの21行目にある「YOUR\_NCMB\_APPLICATION\_KEY」を置き換える
1. 作成したアプリのクライアントキーをコピーして、Monaca IDE上のindex.htmlの22行目にある「YOUR\_NCMB\_CLIENT\_KEY」を置き換える
1. js/favorite-online.jsの「(1) SDKの初期化処理を記述するところ」の部分に以下のコードを記載する

        this.ncmb = new NCMB(options.applicationKey, options.clientKey);


1. js/favorite-online.jsの「(2) 保存先クラスを定義するところ」の部分に以下のコードを記載する

        this.FavoriteClass = this.ncmb.DataStore("favorite");

1. js/favorite-online.jsの「(3) 保存するオブジェクトを生成するところ」の部分に以下のコードを記載する

        var favorite = new this.FavoriteClass();

1. js/favorite-online.jsの「(4) 保存したい値をセットし、保存するところ」の部分に以下のコードを記載する

        favorite.set("uuid", self.uuid)
                .set("url", url)
                .save()
                .then(function(favorite){
                    self.apply(item);
                })
                .catch(function(error){
                    self.apply(item);
                });

1. js/favorite-online.jsの「(5) uuidとurlの両方が合致するオブジェクトを検索し、見つけたものを削除するところ」の部分に以下のコードを記載する

        this.FavoriteClass.equalTo("uuid", self.uuid)
                          .equalTo("url", url)
                          .fetch()
                          .then(function(favorite){
                              favorite.delete()
                                      .then(function(result){
                                          self.apply(item);
                                      })
                                      .catch(function(error){
                                          self.apply(item);
                                      });
                          })
                          .catch(function(error){
                              self.apply(item);
                          });


1. js/favorite-online.jsの「(6) urlだけが合致するオブジェクトの数を取得し、星の横に表示するところ」の部分に以下のコードを記載する

        this.FavoriteClass.equalTo("url", url)
                          .count()
                          .fetchAll()
                          .then(function(results){
                              if (results.count > 0) {
                                  icon.text(results.count);
                              } else {
                                  icon.text("0");
                              }
                          })
                          .catch(function(error){
                              console.log(error.message);
                              icon.text("0");
                          });

1. js/favorite-online.jsの「(7) urlとuuidの両方が合致するオブジェクトの数を取得し、星の色を変更するところ」の部分に以下のコードを記載する

        this.FavoriteClass.equalTo("uuid", self.uuid)
                          .equalTo("url", url)
                          .count()
                          .fetchAll()
                          .then(function(results){
                              if (results.count > 0) {
                                  icon.addClass('fa-star');
                                  icon.removeClass('fa-star-o');
                              } else {
                                  icon.removeClass('fa-star');
                                  icon.addClass('fa-star-o');
                              }
                          })
                          .catch(function(error){
                              console.log('own favorite check error: ' + error.message);
                          });

1. [Monacaデバッガー](http://ja.monaca.io/debugger.html)で動作確認をする

## 参考リンク
* Monaca
  * [https://ja.monaca.io/](https://ja.monaca.io/)
* ニフティクラウドmobile backend
  * [http://mb.cloud.nifty.com/](http://mb.cloud.nifty.com/)
* MonacaのRSSリーダーのサンプル
  * [https://github.com/monaca/project-templates/tree/master/2-rss](https://github.com/monaca/project-templates/tree/master/2-rss)
  * [http://docs.monaca.mobi/cur/ja/sampleapp/samples/sample_rss_reader/](http://docs.monaca.mobi/cur/ja/sampleapp/samples/sample_rss_reader/)

