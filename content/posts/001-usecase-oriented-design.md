---
title: "ユーザーストーリーをソフトウェアの設計にそのまま落とし込むアイデア"
date: 2021-04-20T21:56:31+09:00
tags:
  - プロマネ
  - 設計
---

アジャイルな開発を導入する際に、バックログに積むタスクをユーザーストーリーとして表すプラクティスを取り入れることが多い。
シンプルなルールなので取り組みやすいイメージがあるのだけれども、プロジェクトが進んでいくとロールやユーザーができることの抽象度などを運用していくことが思いの外難しいと感じることが個人的には多いのだ。

原因の一つはユーザーストーリーが「タスク内容を分かりやすくまとめたタイトル」という役目に止まっていることなのかもしれない。

もし、ユーザークラスをソフトウェアの機能の単位として扱うことができれば、開発メンバーなどが普段からユーザーストーリーの表現そのものを意識する機会も増えて上記の問題は解決できるのかもしれない。また、ソフトウェアの設計自体もバックログに積まれていたタスクと一対一の対応が取れて、保守がしやすくなる可能性もある。

<!--more-->

```dart:users.dart
class class User {}

class Guest extends User {}

class Member extends User {
    final String id;
}
```

このように、プロダクトを利用するユーザーを全てクラスとして定義する。

それぞれのユーザーストーリーは、何かのユーザーを表現するクラスのメンバーとして定義するのが良さそうだ。

```dart:member_see_their_purchase_history.dart
// member_see_their_purchase_history.dart
extension MemberSeeTheirPurchaseHistory on Member {
    List<dynamic> seeTheirPurchaseHistory() async {
        return await DB.instance().getPurchaseHistoryOf(this.id);
    }
}
```

うまくユーザーストーリーを設計することができれば、このユーザーストーリーが表すのはユースケースやアプリケーションロジックである。

```dart:server.dart
// server.dart
router.get('/purchase-history', (request) async {
    // リクエストなどからユーザーのロールを取得する
    final member = accountFromHeader(request.header);

    // ロールがメンバーであれば、購買履歴を取得する
    if(member is Member) {
        final history = await member.seeTheirPurchaseHistory();
        return jsonEncode(history);
    } else {
        throw 401;
    }
});
```

コード量の関係で、リポジトリが`DB`というクラスであったり、api サーバが json を返したりすることが前提になっているが、実際はこの辺りは依存性の注入などによって切り替えられるようにしておくと良いだろう。
