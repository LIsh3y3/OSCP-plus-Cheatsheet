- [PEN-200 Reporting Requirements](https://help.offsec.com/hc/en-us/articles/360046787731-PEN-200-Reporting-Requirements)に添付されているMicrosoft Wordファイルをダウンロードし、Google Docsにアップロードして編集する
- コードブロックよりもキャプチャがいい
- キャプチャはObsidianに貼っておけば、finderから閲覧できる
- ノートには失敗したコード含め書くべきだが、レポートにはPoCとして成功するコードとその結果を書く（失敗したコードはいらない）
- wordテンプレの1-3章はレポートの説明が記載されてるが、そのまま残してOK

# Google Docsの編集

- 目次は最後に更新！
- 画像はコピペ可能
- コードブロックはアドオンのCode Blocksを使う
	- 拡張機能 > code blocks > start → 右ペインにcode blocksが表示
	- 任意の箇所を選択して、format
- ツール >  設定から以下は無効にしておく
	- 自動スペル修正
	- 単語の先頭大文字
	- スマート　クオート
- レポートは日本語で書き→選択→DeepLで英語に翻訳し、右下の三↑をクリック
![[Pasted image 20251027074852.png]]
- 最後にファイル>ダウンロード>PDFでダウンロード

---

# 以下ムリ！（Pandoc , LateX)

めちゃくちゃ色々試したが、pandocプラグインが全然メンテされていないことや、pandoc標準のlongtableが出力を邪魔したこと、画像ファイルがうまく表示されないことから、この方法は無理だった。同じことをやらないため、記録しておく。

- 参考記事：[Offsec資格のレポート作成を楽にするメモ with Obsidian](https://r1k0t3k1.github.io/note/blog/offsec-report-with-obsidian)
	- Linux環境であれば、この記事を読めば済む
- 使用ツール：[Offensive Security Exam Report Template in Markdown - GitHub](https://github.com/noraj/OSCP-Exam-Report-Template-Markdown?tab=readme-ov-file)

## 環境

- MacOS：14.6.1（23G93）
- パッケージマネージャ：brew

---

## インストール手順

1. ObsidianのコミュニティプラグインであるPandocをインストールし、有効化
	- Obsidianのインストール方法、コミュニティプラグインの有効化手順は、上記参考記事を読む

2. macのターミナルでpandocやその他必要なツールをインストールする
```bash
brew install pandoc
brew install --cask basictex
```

3. インストール後、`PATH`にTeXのコマンドを追加
```bash
ls /usr/local/texlive/
sudo /usr/local/texlive/<lsの出力(2025basic等)>/bin/universal-darwin/tlmgr path add
```

4. LaTeXパッケージの追加
```bash
sudo tlmgr update --self
sudo tlmgr install collection-fontsrecommended collection-latexrecommended collection-latexextra sourcesanspro ly1 sourcecodepro
```

5. Eisvogelテンプレートのインストール
	- [pandoc-latex-template - GitHub](https://github.com/Wandmalfarbe/pandoc-latex-template/releases/)
	- [ ] 2.4.2にしてみた。記事と同じく。バージョンの互換性の可能性あり。
	- [ ] ==互換性の問題かも==
```bash
# 上記リンクからダウンロード
mkdir -p ~/.pandoc/templates/
cd ~/.pandoc/templates/
mv ~/Download/<ファイル> ./eisvogel.latex.zip
unzip eisvogel.latex.zip
cd <unzipの出力フォルダ>
ls -1 |  grep -v -E '^eisvogel.latex$' | xargs rm -rf

mv eisvogel.latex ../eisvogel.latex
cd ../
rm -r <unzipの出力フォルダ> eisvogel.latex.zip
```
- →.pandoc/templates/eisvogel.latex

6. 動作確認→バージョン情報出力で成功
```bash
pandoc --version
pdflatex --version
xelatex --version
```

---

## プラグイン設定手順

1. Pandocプラグインのオプションを開く

2. Pandoc Pathに、以下コマンドをターミナルで実行した出力を記入
```bash
which pandoc
```

3. PDF LaTeX pathに、以下コマンドをターミナルで実行した出力を記入
```bash
which pdflatex
```

4. Extra Pandoc argumentsに以下のオプションを入力
```bash
 --template eisvogel
 --from markdown+yaml_metadata_block+raw_html
 --table-of-contents
 --toc-depth 4
 --number-sections
 --top-level-division=chapter
 --syntax-highlighting breezedark
 --resource-path=.:src
 --syntax-highlighting=idiomatic
```

5. Export files from HTML or markdown?：Markdownを選択

6. Obsidianを再起動

---

## レポート出力方法

1. norajの[Available templates - GitHub](https://github.com/noraj/OSCP-Exam-Report-Template-Markdown?tab=readme-ov-file#available-templates)をダウンロードし、Obsidianのpandocプラグインを有効化したvaultに保存しておく
2. 