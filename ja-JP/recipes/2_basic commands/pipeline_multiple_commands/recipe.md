### 問題

単一のアトミックコマンドを使って、いくつかの Redis コマンドを実行したい。

### 解法

コマンドのキューを作成し、それらをアトミックに実行するには MULTI/EXEC コマンドを使います。
キューの開始には `MULTI` コマンドを使います。Redis は `OK` を返します。
それから各コマンドのキューを作ってください。最終的には、`EXEC` を使用してコマンドを実行します。
Redis は各コマンド毎にか返値と一緒に multi-bulk を返します。
この基本的な例で、3つの値をリストに追加し、'country-count' と呼ばれるキーを（3に）増加させ、
それからリストのすべての値の範囲を求めます。

必要なら、MULTI キュー と既存のキューをクリアするために DISCARD コマンドを使うことができます。

	redis> MULTI
	OK
	redis> LPUSH country_list france 
	QUEUED
	redis> LPUSH country_list italy
	QUEUED
	redis> LPUSH country_list germany
	QUEUED
	redis> INCRBY country_count 3
	QUEUED
	redis> LRANGE country_list 0 -1
	QUEUED
	redis> EXEC
	1. (integer) 1
	2. (integer) 2
	3. (integer) 3
	4. (integer) 3
	5. 
	 1. germany
	 2. italy
	 3. france
	 

### 検討

MULTI/EXEC （Redis 2.0 で追加予定）は Redis の非常に重要なコンポーネントで、多くを語るに値します。
これが何をするのか、そして何をしないのか知ることはとても重要です。

M/E は次に述べるような意味で、「アトミック」です。
キューが実行されている間、他のクライアントが全く Redis サーバにアクセスできません 
-- このキューは単一操作として扱われます。これはデータの完全性にとって重要です。

M/E プロセスは、コマンドがキューに追加されるときはいつでも、
構文エラーがあるとすぐに構文エラーを報告しながら実行するよう設計されています。例えば：

	redis> MULTI
	OK
	redis> LPUSH country_list italy
	QUEUED
	redis> LPUSH country_list italy germany
	Wrong number of arguments for 'lpush'


さて、 M/E は完全な「トランザクション」を提供 *しません* 
-- 少なくとも常識的な意味では。-- 「ロールバック」の機能を含んでいないからです。
以下のような状況を考えてください。
比較的大きな M/E キュー（例えば、200個のコマンド）を作成して、EXEC を実行します。
すべての溜め込まれたコマンドが実行される前に、
（おそらくメモリを使い果たして）コマンド #148 でサーバがクラッシュしたとしましょう。
はじめの 148 個のコマンドは確実に実行されますが、残りは実行されません。

実際は、M/E では、EXEC コマンドが実行される *前* （つまり、溜め込み中）は 'すべてか無しか' 操作だけなのです。
-- 実行中はそうではありません。 

Redis は問題に対処する興味深い方法を提供します。：馴染み深い追加のみファイルです。
Redis の次バージョンでは、キューのコマンド群は EXEC コマンドが完全に成功したときに AOF に書かれるだけです。
このため、万一サーバが中途半端な EXEC 状態でクラッシュしても、前の pre-EXEC 状態に従って状態を再構築できます。


### 参照




