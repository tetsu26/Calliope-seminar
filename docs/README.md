# README

オープンソースエネルギーモデリングソフト「Calliope（カリオペ）」のB4向け入門資料です。
ご自由にお使いください。

* [1日目資料](chapter_1.md)
* [2日目資料](chapter_2.md)

contact: [Nakata Lab](http://www.eff.most.tohoku.ac.jp/){:target="_blank"}

## この文章のビルド方法

!!! Warning
    conda環境下で```pip```を使うと環境が破壊されてしまう可能性があるので、その場合は[次節の手順](README.md###AnacondaでPythonをインストールしている場合)でインストールする。
    

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

上記コマンドを入力すると自動的にローカルサーバーが立てられ、初期設定では[http://127.0.0.1:8000](http://127.0.0.1:8000){:target="_blank"}にアクセスすれば作成したドキュメントが表示される。ドキュメントを保存すると自動的にビルドされ、更新される。

### AnacondaでPythonをインストールしている場合
```
conda install -c conda-forge mkdocs
conda install -c conda-forge pymdown-extensions
conda install -c conda-forge mkdocs-material
```
でmkdocsをインストールする。**なお、``fontawesome_markdown``だけは``conda``からインストールできない**ので
```
pip install fontawesome_markdown
```
でよい。今のところ、```conda```と```pip```の混用により末松のconda環境では問題は起きていないが、心配な人はmkdocs用に仮想環境を作ってからインストールすれば万一の被害を低減できる。その場合は、
```
conda create -n <任意の仮想環境名>
conda activate <任意の仮想環境名>
```
をパッケージのインストール前に実行すればよい。

[参考にしたサイト](https://qiita.com/mebiusbox2/items/a61d42878266af969e3c){:target="_blank"}