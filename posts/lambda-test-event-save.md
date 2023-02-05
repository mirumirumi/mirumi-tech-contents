---
title: Lambda の共有テストイベント保存時に謎の「Malformed request.」が出る場合、lambda-testevent-schemas の容量オーバーを疑う
tags : [AWS, Lambda]
---

Lambda の共有テストイベント（関数ごとに 10 個までの制限があるほうではなく環境全体で共有できるやつ）を作成して保存するとき、謎の「Malformed request.」というエラーがひたすら出まくってどうしようもないときの解決法。

どうしても解決したかったので Lambda のテストイベントについてなんでもいいからリファレンスを読んでみようということで、とりあえず [このページ](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/testing-functions.html#creating-shareable-events) を漁っていたところ、

> Lambda は、共有可能なテストイベントをスキーマとして lambda-testevent-schemas という名前の Amazon EventBridge (CloudWatch Events) スキーマレジストリに保存します

という記述を見つけ、実際にそれを見に行ってみたところ<strong>まさかのひとつのスキーマの中にすべての共有イベントが複合的に保存されている</strong>ことを知り、なんとなくこれを見て「ひょっとして保存しているテストイベント全体のデータサイズが大きすぎて弾かれている…？」という仮説に繋がりました。

API のテストをやっているときだったので、POST や PUT 系リクエストのテストとして base64 エンコーディングされたデータを扱っていました。テストイベントの文字数が多いのはなんとなく気になっていたのですぐにこの仮説に思い至ったのですが、他のイベントで保存していた base64 部分などを中心に削減してみると…

いままで「Malformed request.」だったものが全く同じ内容で通るようになりました。もっとちゃんとしたエラーを出してくれ…！！頼む…！！！

特に API 系のテストをする場合は「API Gateway AWS Proxy」などのイベントテンプレートを使うと思うんですが、デフォルトの状態だとだいぶ無駄なキーで盛り盛りな感じになってしまっているので、よりこの問題に遭遇しやすいと思われます。エンドポイントごとにテストイベントを作ることにもなるし。