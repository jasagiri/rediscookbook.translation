### 問題

Redis にチャットのようなコミュニケーション（や任意の非同期コミュニケーション）を必要とするアプリケーションのバックエンドとして機能してほしい。

### 解法

Redis 組み込みの `PUBLISH` と `SUBSCRIBE` を使います。これらコマンドはチャンネルのコンセプトを使っています。クライアントはメッセージを受け取るためにどんなチャンネルでも SUBSCRIBE でき、どんなチャンネルのメッセージでも PUBLISH できます。 
チャンネルは１対多の関連を持っています。これは、そのチャンネルで `PUBLISH` されたメッセージはそのチャンネルを購読している人全員が受け取れるということを意味します。

クライアントは `SUBSCRIBE` コマンドを発行すると、`SUBSCRIBE` 以外のコマンドを発行することができず、もはや `UNSUBSCRIBE` されるまでどんなチャンネルも購読されません。
これはアプリケーションがどのように Redis 接続を制御するかについて考えることが重要になることを意味します。
最も簡単なアプローチは1つ以上の `SUBSCRIBE` コマンドを発行したすべてのユーザに Redis 接続を作成することです。

次の例で、`connectionA` と `connectionB` は異なったユーザの異なった接続を表現しています。

    connectionA> SUBSCRIBE room:chatty
    ["subscribe", "room:chatty", 1]

    connectionB> PUBLISH room:chatty "Hello there!"
    (integer) 1

    connectionA> ...
    ["message", "room:chatty", "Hello there!"]

    connectionA> UNSUBSCRIBE room:chatty
    ["unsubscribe", "room:chatty", 0]

`SUBSCRIBE` と `UNSUBSCRIBE` コマンドは常に3要素の配列（実行ログ、操作で影響のあったチャンネル、操作の後も購読のまま残る数）を返します。

**注意**: クライアントはチャンネルを一度購読すると、常に入力メッセージの準備ができているべきです。従って、Redis 接続プール用にイベントループモデルを使うのが最も簡単です。

### 検討

ウェブの環境で、`PUBLISH`/`SUBSCRIBE` を実装しようとすると、最も簡単なアプローチは
[Web Sockets](http://en.wikipedia.org/wiki/Web_Sockets) を利用することです。
WebSockets を利用して、
`PUBLISH`/`SUBSCRIBE` の非同期を自然に行うことができ、すぐに、ユーザ特有の Web Socket のチャンネルに購読されたメッセージを書き込むことができます。

これを書いている時点では、このレシピでみてきたコマンドは新しいです。これらは安定版 Redis リリースでは利用できません。2.0 がリリースされるまで、Redis の開発バージョンを使う必要があります。

### 別検討

#### 永続化されるメッセージ

ウェブチャットのシナリオでは、
`PUBLISH` されたメッセージを永続化したくはありません。これを実装する1つの方法は、リストにメッセージを格納することです。この方法を使った場合、チャンネルに対して`PUBLISH`された最後の `N` メッセージを検索するために `LRANGE` を使うことができます。

    MULTI
    LPUSH room:chatty:backlog (message)
    PUBLISH room:chatty (message)
    EXEC

しかし、より複雑な仕事をこなす必要がありメッセージを送るとすると、sorted set を選ぶことができます。メッセージスタンプはスコアとして使われ、
`ZRANGEBYSCORE` を使って、特定の時間枠のすべてのメッセージを簡単に検索できます。

#### 独裁者

おそらく、あなたはアプリケーションですべてのチャンネルを監視する方法を必要になります。
これを実装する方法は、連絡用の新しいチャンネルを
`SUBSCRIBE` し、使わないチャンネルは
`UNSUBSCRIBE` することです。しかしながら、この方法は高価で、アプリケーションに 
不要なロジックを必要とします。これを解決するには、
`PSUBSCRIBE` コマンドを利用できます。
このコマンドでクライアントはチャンネル識別のパターンを購読できるようになります。
すべてのチャットルームのすべてのメッセージを受け取るには、次のコマンドを発行できます。 

    PSUBSCRIBE room:*

チャンネルを通ってやりとりされたメッセージはチャンネルの名前を含んでいるので、簡単に pattern-subscribe を使って受け取ったメッセージを特定できます。

### 参考

**Atomically Pipeline Multiple Commands** を調べて 
`MULTI`/`EXEC` を使った順序のあるメッセージを事前に永続化する方法を確認してください。

Redis での publish/subscribe の動作例は、 
[this Gist](http://gist.github.com/348262) で確認できます。
