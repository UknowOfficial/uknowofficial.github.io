---
layout: post
title:  "AWS CloudFront x S3 で静的WEBサイトを公開してみる"
date:   2024-05-02
categories: aws
excerpt_image: https://github.com/UknowOfficial/uknowofficial.github.io/blob/main/docs/assets/images/post/2024-05-03-AwsWebsite-009.png?raw=true
tags: [EC2]
---

AWS CloudFrontとS3を使ってWEBサイトを公開してみます。
本ページでは、独自ドメインを使ったHTTPS通信の設定までを実施します。


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [参考サイト：](#参考サイト)
- [STEP1: S3バケットの作成](#step1-s3バケットの作成)
- [STEP2: S3のみでWEBサイト公開](#step2-s3のみでwebサイト公開)
- [STEP3: CloudFront x S3 でWEBサイト公開](#step3-cloudfront-x-s3-でwebサイト公開)
- [独自ドメインを設定](#独自ドメインを設定)
- [接続の確認](#接続の確認)

<!-- /code_chunk_output -->



## 参考サイト：
- [S3のWEBサイトをCloudFrontからのみアクセス可能にする - おかげデザインBlog](https://mik2062.jp/s3-cloudfront/)
- [CloudFrontとS3で作成する静的サイト構成の私的まとめ - DevelopersIO](https://dev.classmethod.jp/articles/s3-cloudfront-static-site-design-patterns-2022/)
- [料金 - Amazon CloudFront - AWS](https://aws.amazon.com/jp/cloudfront/pricing/)


WEBサイトのテンプレートとしてここから持ってきます
[全て無料 レスポンシブホームページテンプレート（全120件）｜ Template Party](https://template-party.com/db_template/?act=list&kind=1&info6=%E5%80%8B%E4%BA%BA%E3%82%B5%E3%82%A4%E3%83%88%E5%90%91%E3%81%91)


## STEP1: S3バケットの作成

設定は以下の感じにしました

リージョン：ap-northeast-1(東京)
パケットタイプ：汎用
パケット名：pp-jp-web-s3-pub-00
ACL:無効
パブリックアクセス：すべてブロック
タグ：とりあえずなし
暗号化：SSE-S3（デフォ）
バケットキー：有効
オブジェクトロック：無効


## STEP2: S3のみでWEBサイト公開

まずは、[S3 静的ウェブサイトホスティング](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/userguide/WebsiteHosting.html)の機能を利用してS3だけでHTMLを公開してみます。ただこのやり方だと、HTTPSをサポートしていないのでHTTPのみ対応となるので、HTTPSを使用したい場合は、次で説明するCloudFrontを使用する必要があります。

1. S3バケット作成
1. 静的ウェブサイトホスティング設定
    - S3バケットを選択
    - プロパティ > 静的ウェブサイトホスティング
    - 有効に変更
1. S3バケットのアクセス設定
    - S3バケットを選択
    - アクセス設定 > ブロックパブリックアクセス (バケット設定)
    - 全てのブロックをオフに設定
    - バケットポリシーの編集
    下記のポリシーを追加
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "PublicReadGetObject",
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::pp-jp-web-s3-pub-00/*"
            }
        ]
    }
    ```
1. S3へHTMLファイルをアップロード
    `index.html`を適当に作成しS3にアップロード
1. サイトへアクセス
    これで "http://pp-jp-web-s3-pub-00.s3-website-ap-northeast-1.amazonaws.com" にアクセスするとindex.htmlの内容が表示されます。

![image001](https://github.com/UknowOfficial/uknowofficial.github.io/blob/main/docs/assets/images/post/2024-05-03-AwsWebsite-001.png?raw=true)

## STEP3: CloudFront x S3 でWEBサイト公開

S3の静的WEBサイトをCloudFrontからアクセスできるようにする手順。独自ドメインが不要であればここまででOKです。

> CloudFront無料利用枠（2024.5.3時点）
> - 1TB のデータ転送 (OUT)
> - 10,000,000 の HTTP または HTTPS リクエスト
> - 2,000,000 の CloudFront Functions の呼び出し
> - 毎月、常に無料

1. S3の設定を変更する
    1. 静的WEBサイトホスティングを無効にする
    1. パブリックアクセスをすべてブロックする
    1. バケットポリシーを削除する（この後変更するので削除する必要はないかも）
1. CloudFrontでディストリビューションを作成
    - オリジンドメインの設定
        `Origin domain` : 対象のS3バケットを選択（プルダウンで選択可能）
    - オリジンアクセス
        `Origin access control settings (recommended)`を選択
        Create new OACで対象S3バケットを指定
        ![image002](https://github.com/UknowOfficial/uknowofficial.github.io/blob/main/docs/assets/images/post/2024-05-03-AwsWebsite-002.png?raw=true)
    - デフォルトのキャッシュビヘイビア
        HTTPS onlyに設定
    - WAF
        一旦WAF設定なしで作成(有料なので)
    - デフォルトルートオブジェクト - オプション
        `index.html`を設定
    - ディストリビューション作成が完了すると、S3バケットポリシーを更新してくださいとアラートが表示されるので、”ポリシーをコピー”ボタンでコピーしておく
1. S3バケットポリシーの変更
    1. 対象S3`pp-jp-web-s3-pub-00` > アクセス許可 > バケットポリシー
    1. 上記でコピーしたポリシーを設定
    コピーした内容は以下
    ```json
    {
            "Version": "2008-10-17",
            "Id": "PolicyForCloudFrontPrivateContent",
            "Statement": [
                {
                    "Sid": "AllowCloudFrontServicePrincipal",
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "cloudfront.amazonaws.com"
                    },
                    "Action": "s3:GetObject",
                    "Resource": "arn:aws:s3:::pp-jp-web-s3-pub-00/*",
                    "Condition": {
                        "StringEquals": {
                        "AWS:SourceArn": "arn:aws:cloudfront::917307172576:distribution/EWJVS5BPMEGZD"
                        }
                    }
                }
            ]
        }
    ```
1. ディストリビューションドメイン名で、ブラウザアクセスするとHTMLの内容が必要されます

![image003](https://github.com/UknowOfficial/uknowofficial.github.io/blob/main/docs/assets/images/post/2024-05-03-AwsWebsite-003.png?raw=true)


## 独自ドメインを設定

お名前.comで取得した独自ドメインでアクセスできるように設定する方法を説明していきます。

参考：
- [S3のWEBサイトを独自ドメインで公開してSSL証明書の自動更新を行う手順 - おかげデザインBlog](https://mik2062.jp/s3-acm/)
- [お名前ドットコムの独自ドメイン名でAWS ACMを使ってSSL証明書を取得する｜基幹システムのクラウド移行・構築・導入支援のBeeX](https://www.beex-inc.com/blog/20231214_onamae_awsacm)

1. お名前.comでドメインを取得
    今回は、無料ドメイン「uknownews.net」を作成しました。
1. Route53を開きます
1. ホストゾーンの作成
    1. ドメイン名を入力
    1. パブリックホストゾーンを選択
    ![image004](https://github.com/UknowOfficial/uknowofficial.github.io/blob/main/docs/assets/images/post/2024-05-03-AwsWebsite-004.png?raw=true)
1. Route53のホストゾーンとお名前.comの独自ドメインとを紐付け
    1. お名前.comにログイン
    1. 対象ドメインを選択し、ネームサーバーの変更を選択
    1. 他のネームサーバーを利用を選択
    1. Route53のタイプNSのレコードに記載される"値/トラフィックのルーティング先”を入力（4つの値）
    ※末尾の「.」は不要です
    1. 登録完了画面が表示されればOK
    （※反映までに24時間程度かかるそう）
    ![image005](https://github.com/UknowOfficial/uknowofficial.github.io/blob/main/docs/assets/images/post/2024-05-03-AwsWebsite-005.png?raw=true)
    1. 名前解決反映の確認
        AWSのネームサーバーで名前解決されていることをコマンドラインから確認します
        ※10分ほどで反映されました
    ```shell
    $ nslookup -type=NS uknownews.net
    Server:		******
    Address:	******

    Non-authoritative answer:
    uknownews.net	nameserver = ns-****.awsdns-**.co.uk.
    uknownews.net	nameserver = ns-****.awsdns-**.com.
    uknownews.net	nameserver = ns-****.awsdns-**.net.
    uknownews.net	nameserver = ns-****.awsdns-**.org.
    ``` 

1. ACM（Amazon Sertificate Manager）でサーバー証明書を発行
    SSL/TLS通信(HTTPS)に必要なサーバ証明書を発行する手続きです。ACMで発行した証明書は無料でCloudFront、ELBで利用する事が可能です。
    1. ACMにアクセス
    1. バージニア北部（us-east-1）にリージョンを変更
    ※証明書は、米国東部 (バージニア北部) リージョン (us-east-1) にある必要があるとCloudFrontの設定画面に記載があり
    1. 証明書をリクエストを選択
    1. パブリック証明書をリクエスト
        - 完全修飾ドメイン名：
            - uknownews.net
            - *.uknownews.net
        - 検証方法：DNS検証
        - キーアルゴリズム：RSA 2048
    ![ACMサーバー証明書](https://github.com/UknowOfficial/uknowofficial.github.io/blob/main/docs/assets/images/post/2024-05-03-AwsWebsite-006.png?raw=true)
    1. Route53でCNAMEレコードの作成
        リクエスト後、作成した証明書のCNAME名とCNAME値が出てきますので「Route53でレコードを作成」を押下します。
    1. しばらくするとACMの対象証明書のステータスが「発行済み」に切り替わります
        ※私の環境では10分ほどかかりました
1. CloudFrontの代替ドメイン（CNAME）設定
    1. CloudFrontで対象ディストリビューションを選択
    1. 代替ドメイン名（CNAME）に下記２つの値を入力
        - uknownews.net
        - www.uknownews.net
    1. Custom SSL certificateで上記ACMで発行した証明書を選択します
    ![image007](https://github.com/UknowOfficial/uknowofficial.github.io/blob/main/docs/assets/images/post/2024-05-03-AwsWebsite-007.png?raw=true)
1. DNSレコードの作成
    1. Route53にアクセス
    1. レコードを作成をクリック
    独自ドメインと紐付けるDNSレコードの作成が必要なため

1. Route53とCloudFrontを紐付け

![image008](https://github.com/UknowOfficial/uknowofficial.github.io/blob/main/docs/assets/images/post/2024-05-03-AwsWebsite-008.png?raw=true)


Route53とは？
AWSが提供するDNSサービスの一つです。AWSでドメインを購入しなくても、お名前.comなど他で取得したドメインを利用できます。
[route53の使い方をわかりやすく　AレコードCNAMEの追加手順 | .LOG](https://log.dot-co.co.jp/route53-a/)

レコードの種類
|種類|説明|備考|
|--|--|--|
|Aレコード|AレコードはドメインとIPアドレスを紐付けるレコード|ドメインを取得して、webサーバの設定を行ってもwebサイトがすぐにみられるわけではありません。ドメインとIPの紐付けが（DNS設定）が必要です。site.example.comにはサーバの住所であるIPアドレスが入っていません。そこで、www.example.com A  IN 203.0.113.14のようにAレコードを設定します。|
|MXレコード|
- CNAMEレコード
- TXTレコード

DNSサーバー（ネームサーバー）とは、ドメイン名とIPアドレスを紐付けるための仕組みを提供するサーバーのことです。
[DNSサーバー（ネームサーバー）とは｜仕組みや設定・確認の方法を解説](https://www.gmo.jp/security/brandsecurity/dns/blog/dns-server/)

## 接続の確認

"uknownews.net" にアクセスしてみるとちゃんとサンプルページが表示されることが確認できました。証明書も有効でちゃんと動作していることも確認ができます。

![image009](https://github.com/UknowOfficial/uknowofficial.github.io/blob/main/docs/assets/images/post/2024-05-03-AwsWebsite-009.png?raw=true)

