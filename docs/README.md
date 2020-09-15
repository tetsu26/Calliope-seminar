# README

オープンソースエネルギーモデリングソフト「Calliope（カリオペ）」のB4向け入門資料です。
ご自由にお使いください。

contact: [Nakata Lab](http://www.eff.most.tohoku.ac.jp/)

## この文章のビルド方法
mkdocsをインストール

```
pip install mkdocs
pip install pymdown-extensions
pip install mkdocs-material
pip install fontawesome_markdown
```

!!! Tip
    祈るとうまくいく

この文章のソースコードを任意の場所にクローンする

```
git clone https://github.com/tetsu26/Calliope-seminar
```

mkdocsでビルド

```
mkdocs build
```

編集しながら作成したドキュメントを確認したいときは、ローカルサーバーを立てたほうが便利

```
mkdocs serve
```

上記コマンドを入力すると自動的にローカルサーバーが立てられ、初期設定では[http://127.0.0.1:8000](http://127.0.0.1:8000)にアクセスすれば作成したドキュメントが表示される。ドキュメントを保存すると自動的にビルドされ、更新される。

[参考にしたサイト](https://qiita.com/mebiusbox2/items/a61d42878266af969e3c)