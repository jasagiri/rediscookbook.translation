### 問題

任意のオブジェクトに重複しない固有の ID を提供したい。

### 解法

Redis 組み込みのアトミックな INCR 関数を利用してください。

	$redis-cli INCR <an_object_name>
	(interger) 1

	$redis-cli INCR <another_object_name>
	(interger) 2

	$redis-cli GET <an_object_name>
	1
		
	$redis-cli GET <another_object_name>
	2
	
### 検討

固有の ID を提供する INCR を使うことは Redis の中心概念の１つです。しばしば「主キー」スタイルとしてリレーショナルデータベースで使われてきた同等な機能の代わりに使用されます。

固有の ID を使うより多くの例は *Redis における Ruby オブジェクトの保存* を参照してください。

### 参照
