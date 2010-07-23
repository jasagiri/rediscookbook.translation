### 問題

任意の Ruby オブジェクトを Redis に保存したい。

### 解法

どんなキー/値データベースでも、構造化するのにキーを使えます。：
ここでは Ruby を使った例ですが、原則どんなプログラミング言語でも利用できます。

    >> redis.set "event:42:name", "Redis Meetup"
    => "OK"

    >> redis.get "event:42:name"
    => "Redis Meetup"

同じ例ですが、今回はまず固有の ID を生成しています。：

    >> id = redis.incr "event"
    => 1

    >> redis.set "event:#{id}:name", "Redis Meetup"
    => "OK"

    >> redis.get "event:#{id}:name"
    => "Redis Meetup"

別解は、保存するときにデータをシリアライズして、検索するときにデコードすることです。：

    >> id = redis.incr "event"
    => 2

    >> redis.set "event:#{id}", {:name => "Redis Meetup"}.to_json
    => "OK"

    >> JSON.parse redis.get("event:#{id}")
    => {"name" => "Redis Meetup"}

最新バージョンの Redis で利用可能なもう１つの解は、新しいデータ Hash 型を使うことです。：

    >> id = redis.incr "event"
    => 3

    >> redis.hset "event:#{id}", "name", "Redis Meetup"
    => "OK"

    >> redis.hget "event:#{id}", "name"
    => "Redis Meetup"

ご覧のように、Redis は非常に柔軟で、情報を保存する最も良い戦略を決められます。

オブジェクト属性に基づくキー生成を自動化するいくつかのライブラリがあります。これらをチェックして使い方を学んでください。

* [DataMapper Adapter](http://github.com/whoahbot/dm-redis-adapter)
* [Ohm](http://ohm.keyvalue.org)
* [Redis Model](http://github.com/voloko/redis-model)
* [Redis Objects](http://github.com/nateware/redis-objects)
* [Remodel](http://github.com/tlossen/remodel)

### 参照

いろいろな戦略的なライブラリを使った Ruby のオプションを確認するには
 **using_a_ruby_library_to_store_objects** をチェックアウトしてください。
