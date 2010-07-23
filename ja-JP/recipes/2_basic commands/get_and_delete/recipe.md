### 問題

Redis からオブジェクトをアトミックに GET し、 DELETE したい。

### 解法

Redis 組み込みのアトミック関数と MULTI-EXEC 関数を使います。

この問題へのアプローチはたぶんこんな風です。中間コード：

    success = RENAME key key:tmp
    if success
      value = GET key:tmp
      DELETE key:tmp
      return value
    end

これは単純ですが、Redis のアトミックな特徴をよく使っています。
RENAME 関数は最初に呼ばれたときに成功しますが、その後呼ばれても key はすでに名前は変わっているので失敗するでしょう。
GET と DEL は RENAME 関数のおかげで、操作中のオブジェクトデータを他のクライアントが参照するのを防げるのです。


しかし、潜在的な問題があります。もし実行が２行目(if success)や３行目(value = GET key:tmp)で中断したとすると、その key はデータベース上「key:tmp」と永遠に改名されたままです。

Redis では MULTI-EXEC 関数を使った解決法が提供されています。

まず、擬似コードで

	MULTI
	value = GET key
	DELETE key
	EXEC
	return value

それから「redis-cli」を使います。

	redis> SET TOTO 1
	OK
        redis> GET TOTO
        1
        redis> MULTI
        OK
        >> GET TOTO
        QUEUED
        >> DEL TOTO
	QUEUED
	redis> EXEC
	1. 1
	2. (integer) 1
        redis> GET TOTO
        (nil)
	
### 検討

### 参照

**Atomically Pipeline Multiple Commands** に MULTI/EXEC のより多くの情報があります。
