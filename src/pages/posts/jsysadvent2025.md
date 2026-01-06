---
layout: ../../layouts/MarkdownPostLayout.astro
title: 'Golang・バトル'
author: 'temari1013'
description: 'あなたとGolang、今すぐダウンロード'
image:
  url: '/arc.png'
  alt: 'The Astro logo on a dark background with a purple gradient arc.'
pubDate: 2025-12-31
tags: ['astro', 'blogging','go','PC','jsys','tsukuba','sohosai' ]
---

## GOlangバトル

# 0.これは何?
これは[jsys advent carender](後でリンク) n 日目の記事です。遅刻して申し訳ありません。
筑波大学の雙峰祭では、[企画検索システム](https://search.sohosai.com/)を作成、公開しています。\

今年は、
1. 企画を場所、企画名、企画タグなどから検索できる
2. 企画をお気に入りできる
3. 企画グランプリに投票できる
という機能を実装することが確定され、私はこれのバックエンド担当の一人として携わることになりました。\

この記事は、その中で出会ったgolangのよくわからない仕様を始めとして、バトルになった場所の解説をします。\

 
# 1.enum型がない
golangには何故か、enum型がありません。\
雙峰祭には企画スタイルという概念があり、ここには事前に決まっている値しか入らないことがわかっていたのでenumを使いたかったですし、実際にDB側ではenumで定義していたのですが、ないものはないので仕方ありません。\
結局、
```
type ProjectStyle string

const (
    ProjectStyleOther         ProjectStyle = "other"
    ProjectStyleFood          ProjectStyle = "food"
    ProjectStyleSweets        ProjectStyle = "sweets"
    ProjectStyleDrink         ProjectStyle = "drink"
    ProjectStyleGoods         ProjectStyle = "goods"
    ProjectStyleParticipatory ProjectStyle = "participatory"
    ProjectStyleExhibition    ProjectStyle = "exhibition"
    ProjectStyleMusic         ProjectStyle = "music"
    ProjectStylePerformance   ProjectStyle = "performance"
)
```
のように定義したあと、
# 2.required

### コンテクスト
今年の企画グランプリでは、企画検索システム内の投票ページにグーグルアカウントでログインし、(ここの詳しい話は、共にバックエンド担当であったsakueniが[記事]()にしています。)
- お気に入りの企画を最大3つ
- 学生かどうか(自己申告)
を解答し、HTTPリクエストとして送信していました。

ところが、今年の企画投票では、初日の午前まで学生以外が投票できない、という大きめのバグを出してしまっており、これは、投票のHTTPリクエストを以下のように定義した構造体にバインディングしていたことに起因していました。。

 ```
 type VoteRequest struct {
	Projects  []ProjectRepresentation `json:"projects" binding:"required"`
	IsStudent bool                    `json:"is_student" binding:"required"`
}
  ```

問題となっているのは `IsStudent bool                    json:"is_student" binding:"required"` の行で、おそらくIsStudentの値がundifined/nullのリクエストを弾くために設定されたものですが、結果として `"is_student":false`のリクエストをすべて弾いてしまっていました。\
あくまでGolangの記事なので、この件の事後対応には触れませんが、それなりの対応が必要となりました。\




