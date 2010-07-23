### 問題

基本的な push と pop 操作でシンプルで抽象化された、先入れ/先出しのキューを実装して Redis を使いたい。

### 解法

Redis 組み込みの `List` データ型は自然なキューです。事実、単なるキューの実装で、`List` 操作の限定されたセットを便利して使うだけです。

	redis> LPUSH queue1 tom
	(integer) 1
	redis> LPUSH queue1 dick
	(integer) 2
	redis> LPUSH queue1 harry
	(integer) 3
	redis> RPOP queue1
	tom
	redis> RPOP queue1
	dick
	redis> RPOP queue1
	harry


### 検討

Redis は *ブロッキングポップ*操作と同等の４つの基本的なリストプッシュとポップ操作(RPUSH, LPUSH, LPOP, RPOP)が提供されています。これらはすべて O(1) オーダで、コマンドの時間的コストはリストの長さに依存しません。

Redis コマンド上で簡単なキューを実装するには、簡単で、さらっと Redis の力を把握するには良いイントロです。

例えば、ここに、オブジェクトレベルの相互作用（新しいキューの ID を一意にするために INCR を使います）を提供する Python キューがあります。: 

    r = redis.Redis()

    class Queue(object):
        """抽象 FIFO キュー"""
        def __init__(self):
            local_id = r.incr("queue_space")
            id_name = "queue:%s" %(local_id)
            self.id_name = id_name
 
    def push(self, element):
        """キューの最後に要素を Push する""" 
            id_name = self.id_name
            push_element = redis.lpush(id_name, element)
 
    def pop(self):
        """キューの最初の要素を Pop する。"""
            id_name = self.id_name
            popped_element = redis.rpop(id_name)
            return popped_element


### 参照

