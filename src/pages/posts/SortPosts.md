---
layout: ../../layouts/MarkdownPostLayout.astro
title: '記事を投稿日ソート'
author: 'temari'
description: '投稿が日付順になるように調整しました。'
image:
  url: '/arc.png'
  alt: 'The Astro logo on a dark background with a purple gradient arc.'
pubDate: 2025-04-11
tags: ['astro', 'feature']
---

## 記事を日付降順になるようにソートしました。

インターネットに、Dateへのパースは実行環境に依存すると脅されたのであたたかみのある手書き関数でパースしました

```
function parseDate(str :string){
   return parseInt( str.replace(/-/g, ''), 10);
};

const sortedPosts =  allPosts.sort((a,b) =>{
const dateA =parseDate(a.frontmatter.pubDate);
const dateB =parseDate(b.frontmatter.pubDate);
return dateB - dateA;
});
```

春だし、あたたかいとうれしい。
