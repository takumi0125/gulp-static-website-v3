gulp-static-website-v3
===============================

静的サイト制作用の汎用gulpタスクテンプレートです。

## インストール
```bash
mkdir yourProject
cd yourProject
git clone --recursive git@github.com:takumi0125/gulp-static-website-v3.git .
cd gulp
npm install
```

## 概要

`gulp` コマンドで `gulp/src/` の中身がタスクで処理され、ディレクトリ構造を保ちつつ `htdocs/` に展開されます。ただし、「 _ (アンダースコア) 」で始まるファイルやディレクトリはコンパイル・コピーの対象外です。スプライト用のソース画像を格納するディレクトリや、Sassで@import するファイルは「 _ (アンダースコア) 」をつけておけば、 `htdocs/` に展開されることはありません。

`gulp watcher` コマンドでローカルサーバが立ち上がります。実行中は
```
http://localhost:50000/
```
で展開後のページが確認できます。

※ <a href="https://github.com/assemble/assemble" target="_blank">assemble</a> タスクも定義されていますが、<a href="https://github.com/assemble/gulp-assemble" target="_blank">プラグイン</a>がアルファ版なので、正しく動作しない可能性があります。

`src/_data.json` は jade や assemble のタスクを実行する際に読み込まれます。
メタ情報等を定義しておけば、_data.jsonファイルで一元管理できます。

## 主要タスク

```
gulp
```
`gulp/src/` の中身を各種タスクで処理し `htdocs/` に展開します。
`--rerealse` オプションを指定すると、 JS ファイルを minify します。

```
gulp watcher
```
ディレクトリを監視し、変更があった場合適宜タスクを実行します。また、ローカルサーバを立ち上げます。
SSI インクルードに対応しています。

```
gulp bower
```
`bower.json` で定義されているJSライブラリを `htdocs/assets/_lib/` にインストールします。開発開始時に実行して下さい。


その他のタスクは `gulpfile.coffee` をご参照ください。


## cleanタスクについて
デフォルトタスクを実行、または、

```
gulp clean
```

でクリーンタスクが実行されます。
クリーンタスクの対象ディレクトリは

```coffeescript
# clean対象のディレクトリ (除外したいパスがある場合にnode-globのシンタックスで指定)
CLEAN_DIR = [ "#{PUBLISH_DIR}/**/*" ]
```

このように指定してあります。除外したいディレクトリやファイルがある場合は、こちらに設定を追加してください。


## 個別タスク生成用関数

スプライト画像、browserify, jsのconcat, coffee scriptのconcatを使用する場合は、以下の関数を使用し、タスクを定義してください。
以下の関数を実行すると、watchタスクも同時に定義されます。

個別タスクは

```
#################
### 個別タスク ###
#################
```

と

```
### 個別タスクここまで ###
```

の間に記述してください。


※ `SRC_DIR` はソースファイルの格納ディレクトリです。


### createSpritesTask

spritesmith のタスクを生成する関数です。

※ スプライト生成には <a href="https://github.com/twolfson/gulp.spritesmith" target="_blank"> gulp.spritesmith</a> を使用しています。<br>
※ 画像の圧縮には <a href="https://github.com/imagemin/imagemin-pngquant" target="_blank">imagemin-pngquant</a> を使用しています。

**Params**

 - `taskName` **{Object}**: タスクを識別するための名前 すべてのタスク名と異なるものにする
 - `cssDir` **{String}**: ソース画像ディレクトリへのパス ( `SRC_DIR` からの相対パス)
 - `imgDir` **{String}**: ソース CSS ディレクトリへのパス ( `SRC_DIR` からの相対パス)
 - `outputImgName` **{String}**: CSS に記述される画像パス (相対パスの際に指定する)
 - `outputImgPath` **{String}**: 指定しなければ #{taskName}.png になる
 - `compressImg` **{Boolean}**: 画像を圧縮するかどうか

```coffeescript
"#{SRC_DIR}#{imgDir}/_#{taskName}/"
```
以下にソース画像を格納しておくと
```coffeescript
"#{SRC_DIR}#{cssDir}/_#{taskName}.scss"
```
と
```coffeescript
"#{SRC_DIR}#{imgDir}/#{outputImgName or taskName}.png"
```
が生成されます。

### createCoffeeExtractTask

coffee script で concat する場合のタスクを生成する関数です。

**Params**

 - `taskName` **{Object}**: タスクを識別するための名前 すべてのタスク名と異なるものにする
 - `src` **{Array|String}**: ソースパス node-glob のシンタックスで指定
 - `outputDir` **{String}**: 最終的に出力される js が格納されるディレクトリ
 - `outputFileName` **{String}**: 最終的に出力される js ファイル名(拡張子なし)

### createBrowserifyTask
browserify のタスクを生成する関数です。coffee script と bowerを使用できます。

bowerを使用する場合は、タスク実行前に
```
bower install
```
を実行してください。

※ 引数に `src` entries　を除いた全ソースファイルを指定している理由は、coffeeInclude の watchタスクから除外するためです。

**Params**

 - `taskName` **{Object}**: タスクを識別するための名前 すべてのタスク名と異なるものにする
 - `entries` **{Array|String}**: browserify の entriesオプションに渡す node-glob のシンタックスで指定
 - `src` **{Array|String}**: entries を除いた全ソースファイル ( watch タスクで監視するため) node-glob のシンタックスで指定
 - `outputDir` **{String}**: 最終的に出力されるjsが格納されるディレクトリ
 - `outputFileName` **{String}**: 最終的に出力される js ファイル名(拡張子なし)


### createJsConcatTask

javascriptのconcatタスクを生成する関数です。ソースは minify されます。

**Params**

 - `taskName` **{Object}**: タスクを識別するための名前 すべてのタスク名と異なるものにする
 - `src` **{Array|String}**: ソースパス node-glob のシンタックスで指定
 - `outputDir` **{String}**: 最終的に出力される js が格納されるディレクトリ
 - `outputFileName` **{String}**: 最終的に出力される js ファイル名(拡張子なし)
