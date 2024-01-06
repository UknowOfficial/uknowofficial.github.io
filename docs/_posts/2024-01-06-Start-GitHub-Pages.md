---
layout: post
title:  "Start Gihub Pages"
date:   2024-01-06 09:24:13 +0900
categories: github-pages
excerpt_image: 
tags: [setup]
---

GitHub Pages x Jekyllのセットアップ方法に関する記事です。Jekyllテーマとして[jeffreytse/jekyll-theme-yat](https://github.com/jeffreytse/jekyll-theme-yat/tree/master)を利用して進めていきます。`Yat`テーマは、デザインがシンプルで使いやすお、カテゴリやタグ分け機能が入っており記事投稿に向いたサイトが簡単に作成出来るところが良いところかなと思います。

このページでは、以下が出来るようになることを目指します。

- GitHub Pages x Jekyllのセットアップ（既存Jekyllテーマ利用）
- ホーム画面のカスタマイズ
- 記事の投稿

サイトイメージ
![github-pages-image](https://github.com/UknowOfficial/uknowofficial.github.io/blob/main/docs/assets/images/post/2024-01-06-GithubPages-003.png?raw=true)


---


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [GitHub Pages x Jekyllのセットアップ](#github-pages-x-jekyllのセットアップ)
  - [リポジトリを作成](#リポジトリを作成)
  - [Collaboratorsの設定](#collaboratorsの設定)
  - [Jekyllでページレイアウト設定](#jekyllでページレイアウト設定)
    - [Ruby開発環境のセットアップ](#ruby開発環境のセットアップ)
    - [Jekyllのインストール](#jekyllのインストール)
    - [リポジトリをローカルにclone](#リポジトリをローカルにclone)
    - [Gemfileの修正](#gemfileの修正)
    - [_config.ymlの修正](#_configymlの修正)
    - [必要フォルダ・ファイルの作成](#必要フォルダファイルの作成)
- [ホーム画面のカスタマイズ](#ホーム画面のカスタマイズ)
  - [タイトルの変更](#タイトルの変更)
  - [フッターの修正](#フッターの修正)
  - [ホーム画像の変更](#ホーム画像の変更)
  - [リンク用にSNSアイコンなどを表示させたい](#リンク用にsnsアイコンなどを表示させたい)

<!-- /code_chunk_output -->




## GitHub Pages x Jekyllのセットアップ

GitHub Pagesの公式ページ「[GitHub Pages サイトを作成する](https://docs.github.com/ja/pages/getting-started-with-github-pages/creating-a-github-pages-site)」を参考にしながら進めていきます。  
※今回は無料アカウントなのでパブリックリポジトリで設定します

> 注意事項
GitHub Pagesは、GitHub Free及びOrganizationのGitHub Freeのパブリックリポジトリ、GitHub Pro、GitHub Team、GitHub Enterprise Cloud、GitHub Enterprise Serverのパブリック及びプライベートリポジトリで利用できます。

### リポジトリを作成

1. `<user>.github.io` または `<organization>.github.io` という名前のリポジトリを新規作成します。  
    今回は次のリポジトリ名で設定しました:  
    >  `uknowofficial.github.io`

    * リポジトリ名は小文字である必要があります
    * アカウントが GitHub Free （Organization 用含む）を使用している場合、Publicリポジトリである必要があります
  
1. [Initialize this repository with a README] (このレポジトリを README で初期化する) を選択し、リポジトリを作成します  
  
1. GitHub Pagesサイトの公開設定
mainリポジトリにPushされたことをトリガにページを公開するように設定します。ディレクトリを管理しやすいように、「/docs」以下を公開対象にします。`{リポジトリ} > Setting > Pages`から以下のように設定します。
    - Source : Deploy from a branch
    - Branch : main
    - Folder : /docs


### Collaboratorsの設定

`UknowOfficial/uknowofficial.github.io`のリポジトリ設定で、各メンバーが各自のGithubアカウントから編集できるように招待します。

- `{リポジトリ} > Setting > Collaborators > Manage access`からgithubアカウントを追加

> Reference:
> [新しいMacでGitHubのSSH接続をするまでの環境構築手順 #Git - Qiita](https://qiita.com/unsoluble_sugar/items/14bea376d8e6fce82eb3)


### Jekyllでページレイアウト設定

Jekyllを使ってサイトの見た目をカスタマイズすることができます。[jeffreytse/jekyll-theme-yat](https://github.com/jeffreytse/jekyll-theme-yat/tree/master)のテーマを利用して進めていきます。

#### Ruby開発環境のセットアップ

[Jekyll公式サイト](https://jekyllrb-ja.github.io/docs/installation/)からOS環境を選択して、Ruby環境を設定してください

> * すでにRubyがインストールされている場合は不要です。
> * `$ ruby -v` で`2.5.0`以上のバージョンであればOKです。
> * Rubyは、rbenvでバージョン管理すると便利です。rbenvでのセットアップ方法は「[rbenvでrubyのバージョンを管理する #Ruby - Qiita](https://qiita.com/hujuu/items/3d600f2b2384c145ad12)」を参照してください

#### Jekyllのインストール

jekyllとbundler gemsをインストールします。

```bash
$ sudo gem install jekyll bundler
```

#### リポジトリをローカルにclone

[リポジトリを作成](#リポジトリを作成)で作成した、リポジトリをローカルにcloneして、必要ファイルを修正しつつ、bundleしていきます。

```bash
$ git clone git@github.com:UknowOfficial/uknowofficial.github.io.git #SSH接続の場合
$ cd uknowofficial.github.io
$ mkdir docs # 各種ファイルを格納するディレクトリを作成
$ cd docs
$ jekyll new --skip-bundle .
```

#### Gemfileの修正

上記で生成されたGemfileを編集し、github-pagesに対応させる
変更箇所は以下：  
（それ以外の項目はとりあえずそのままでOK）

```ruby
# 下記項目のコメントアウトを外す
gem "github-pages", group: :jekyll_plugins
# 依存関係エラーを防ぐために最低バージョンを3.9.3にしておく
gem "jekyll", "~> 3.9.3"
# プラグインにjekyll-theme-yatを追加
group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.12"
  gem "jekyll-theme-yat"
end
```

#### _config.ymlの修正

下記内容に編集します

```yml

# Theme setting
remote_theme: "jeffreytse/jekyll-theme-yat"
minimal_mistakes_skin    : "default" 

# Site settings
title: Uknow Official Site
email: uknow.fam@gmail.com
author: Uknow
copyright: "(cleft) {currentYear} {author}"

description: >- # this means to ignore newlines until "baseurl:"
  This site is Uknow official page.

baseurl: "/" # the subpath of your site, e.g. /blog
url: "https://uknowofficial.github.io/" # the base hostname & protocol for your site, e.g. http://example.com
github_username:  UknowOfficial
search: true

# Build settings
plugins:
  - jekyll-feed
  - jekyll-yat
```

#### 必要フォルダ・ファイルの作成

[jeffreytse/jekyll-theme-yatのリポジトリ](https://github.com/jeffreytse/jekyll-theme-yat/tree/master)をCloneしてきて、`_data`,`_layouts`, `_includes`,`_sass`,`assets`をフォルダごとそのまま、`uknowofficial.github.io/docs`以下にコピーします。中身はこの後適宜変更していきますので、とりあえず全てそのままでOKです。

以下の構成になると思います。

```bash
.
├── _config.yml
├── _data
├── _includes
├── _layouts
├── _posts
│   ├── 2024-01-06-article01.md # ここに記事を追加していきます
│   └── 2024-01-06-article02.md # ここに記事を追加していきます
├── _sass
├── assets
├── _config.yml
├── Gemfile
└── index.md
```

## ホーム画面のカスタマイズ

### タイトルの変更

1. ページタブに表示されるタイトルの変更

index.markdown
```md
title: Home
```

_config.yml
```yml
title: Uknow Official Site
```

![](https://github.com/UknowOfficial/uknowofficial.github.io/blob/main/docs/assets/images/post/2024-01-06-GithubPages-001.png?raw=true)


1. TOPページ内のタイトル, サブタイトルの変更

`docs/_data/defaults.yml`を任意の内容に修正

![](https://github.com/UknowOfficial/uknowofficial.github.io/blob/main/docs/assets/images/post/2024-01-06-GithubPages-002.png?raw=true)



### フッターの修正

`_includes/views/footer.html`を任意の内容に修正

コピーライトは、_config.ymlに下記を追加
※期間で表示したい場合は、2023-{currentYear}のように記述ください
```yml
copyright: "(cleft) {currentYear} {author}"
```


### ホーム画像の変更

ホーム画面のバナー画像を挿入する場合、以下の構成で画像ファイルを格納し、`docs/_data/defaults.yml`￥の`banner`パラメータで相対パスを指定してあげることで設定できます。

```
.
├── assets
    ├── images
        ├── banners
            └── home.jpeg
```

### リンク用にSNSアイコンなどを表示させたい

[Octicons - Primer](https://primer.style/foundations/icons)を利用できます。OctiｃonsはGitHubのサイト内で利用されている様々なアイコンを提供しているサイトです。利用したいアイコンの「Copy SVG」ボタンをクリックして、SVG形式のタグを取得できるので、HTML内の任意の場所にペーストすればそのまま利用できます。


