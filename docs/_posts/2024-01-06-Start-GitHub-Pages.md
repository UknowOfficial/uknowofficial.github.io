---
layout: post
title:  "Start Gihub Pages"
date:   2024-01-06 09:24:13 +0900
categories: github-pages setup
---


# 最初にやること

Gemfileの修正
github-pagesに対応させる
- gem "github-pages", group: :jekyll_plugins
依存関係エラーを防ぐために最低バージョンを3.9.3にしておく
- gem "jekyll", "~> 3.9.3"

_config.ymlの修正

最低限の必要フォルダ・ファイルの作成

jekyll-theme-yatのリポジトリをCloneしてきて、`_data`,`_layouts`, `_includes`,`_sass`,`assets`をフォルダごとそのままコピーして使います。中身はこの後適宜変更していきますので、とりあえず全てそのままでOKです。

[jeffreytse/jekyll-theme-yat: 🎨 Yet another theme for elegant writers with modern flat style and beautiful night/dark mode.](https://github.com/jeffreytse/jekyll-theme-yat/tree/master)



# フォルダ構成

以下の構成でフォルダを作成しておく。

```
.
├── _config.yml
├── _data
│   └── members.yml
├── _drafts
│   ├── begin-with-the-crazy-ideas.md
│   └── on-simplicity-in-technology.md
├── _includes
│   ├── footer.html
│   └── header.html
├── _layouts
│   ├── default.html
│   └── post.html
├── _posts
│   ├── 2007-10-29-why-every-programmer-should-play-nethack.md
│   └── 2009-04-26-barcamp-boston-4-roundup.md
├── _sass
│   ├── _base.scss
│   └── _layout.scss
├── assets
├── _site
├── .jekyll-metadata
└── index.html # can also be an 'index.md' with valid front matter
```

# ホーム画面のカスタマイズ

## タイトルの変更

1. ページタブに表示されるタイトルの変更

index.markdown
```md
title: Home
```

_config.yml
```yml
title: Uknow Official Site
```

![](/assets/images/post/2024-01-06-GithubPages-001.png)


1. TOPページ内のタイトル, サブタイトルの変更

`docs/_data/defaults.yml`を任意の内容に修正

![](/assets/images/post/2024-01-06-GithubPages-002.png)



## フッターの修正

`_includes/views/footer.html`を任意の内容に修正

コピーライトは、_config.ymlに下記を追加
※期間で表示したい場合は、2023-{currentYear}のように記述ください
```yml
copyright: "(cleft) {currentYear} {author}"
```


## ホーム画像の変更

ホーム画面のバナー画像を挿入する場合、以下の構成で画像ファイルを格納し、`docs/_data/defaults.yml`￥の`banner`パラメータで相対パスを指定してあげることで設定できます。

```
.
├── assets
    ├── images
        ├── banners
            └── home.jpeg
```

## リンク用にSNSアイコンなどを表示させたい

[Octicons | Primer](https://primer.style/foundations/icons)を利用できます。OctiｃonsはGitHubのサイト内で利用されている様々なアイコンを提供しているサイトです。利用したいアイコンの「Copy SVG」ボタンをクリックして、SVG形式のタグを取得できるので、HTML内の任意の場所にペーストすればそのまま利用できます。


