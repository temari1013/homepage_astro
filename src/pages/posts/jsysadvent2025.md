---
layout: ../../layouts/MarkdownPostLayout.astro
title: 'Golang・バトル'
author: 'temari1013'
description: 'あなたとGolang、今すぐダウンロード'
image:
  url: '/arc.png'
  alt: 'The Astro logo on a dark background with a purple gradient arc.'
pubDate: 2026-01-06
tags: ['astro', 'blogging','go','PC','jsys','tsukuba','sohosai' ]
---

## Golang・バトル

### 0.これは何?
あけましておめでとうございます。
これは[jsys advent calender](https://adventar.org/calendars/11933)15日目の記事です。遅刻してすみません。
さて、筑波大学の雙峰祭では、[企画検索システム](https://search.sohosai.com/)を作成、公開しています。

今年は、
1. 企画を場所、企画名、企画タグなどから検索できる
2. 企画をお気に入りできる
3. 企画グランプリに投票できる

という機能を実装することが確定され、私はこれのバックエンド担当の一人として携わることになりました。
この記事は、その中で出会ったgolangのよくわからない仕様や、バトルになった場所の解説をします。

 
### 1.enum型がない
golangには何故か、enum型がありません。
雙峰祭には企画スタイルという概念があり、ここには事前に決まっている値しか入らないことがわかっていたのでenumを使いたかったですし、実際にDB側ではenumで定義していたのですが、ないものはないので仕方ありません。
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
のように定義し、バリデーションする関数を作成するか、
```
type AppError struct {
	Message string
	Kind    int
}

const (
	InvalidFormat = iota
	Unauthorized
	VoteCannotBeChanged
	ProjectIsNotVoteTarget
	ProjectNotFound
	TooManyVotes
	OutOfVotePeriod
	DBError
	PermissionDenied
	BookmarkNotFound
	BookmarkAlreadyExists
	TooFewVotesToDecideRanking
)
```
のように定義する必要があります。

### 2.required

- コンテクスト
今年の企画グランプリでは、企画検索システム内の投票ページにグーグルアカウントでログインし、(ここの詳しい話は、共にバックエンド担当であったsakueniが[記事](https://sakueni-blog.pages.dev/posts/20260101_jwt_advent/)にしています。)
 - お気に入りの企画を最大3つ
 - 学生かどうか(自己申告)

を回答し、HTTPリクエストとして送信していました。

ところが、今年の企画投票では、学生以外が投票できない、というバグを出してしまっており、これは、投票のHTTPリクエストを以下のように定義した構造体にバインディングしていたことに起因していました。

 ```
 type VoteRequest struct {
	Projects  []ProjectRepresentation `json:"projects" binding:"required"`
	IsStudent bool                    `json:"is_student" binding:"required"`
}
  ```

問題となっているのは
 ```
 IsStudent bool  `json:"is_student" binding:"required"`
 ```
  の行で、おそらくIsStudentの値がundefined/nullのリクエストを弾くために設定されたものですが、結果として "is_student":falseのリクエストをすべて弾いてしまっていました。一見ここは正しい記述をしているように見えたこともあり、問題に気づくのがかなり遅くなりました。

また、修正時にも、たんにrequiredを消したのでは今度はIsStudentに何も設定されていなくても受け入れてしまうようになり、(学生かどうかが必要であることが確定したのがかなり遅い時期であったことも相まって)今度はすべての投票のIsStudentの値がDB側に入力されておらず、DB側が勝手に全員 IsStudentを falseとしていたことが表面化しました。

本質的には*boolで宣言するように修正などが良かったのでしょうが、当日に発覚したこともあり、場当たり的な修正を行いました。
あくまでGolangの記事なので、この件の事後対応の詳細には触れませんが、それなりの不具合を出してしまったため、それなりの対応が必要となりました。

### 3. 終わりに
これ以外にもバグや、不具合をいくつか出してしまい、それに関しては申し訳ないと同時に、アクセス統計を見る限りかなり多くの人が検索システムを活用してくれていたようで、ありがたい限りです。

Golangは、ここまで述べたようなよくわからない仕様や、特徴的な記法もいくつかありましたが、その上でバックエンドを初めて書く自分でも割り当てられた仕事の分は一応書き切ることができ、良かったです。








