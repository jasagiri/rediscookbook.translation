### 問題 
オブジェクトにタグ付けして、検索できるようにしたい。

### 解法 
この場合は、本を例に使い、タグ付けに SET を1つ作成し、正しいセットを ids に関連付けます。タグ間の組み合わせを選択するには、SINTER、SUNION、SDIFF を使うと簡単です。「タグ」として単語や語幹を使用すると文書保存にも同じ原則を使用できます。

redis-cli にて:

    SET book:1 {'title' : 'Diving into Python',
    'author': 'Mark Pilgrim'}
    SET book:2 { 'title' : 'Programing Erlang',
    'author': 'Joe Armstrong'}
    SET book:3 { 'title' : 'Programing in Haskell',
    'author': 'Graham Hutton'}

    SADD tag:python 1
    SADD tag:erlang 2
    SADD tag:haskell 3
    SADD tag:programming 1 2 3
    SADD tag computing 1 2 3
    SADD tag:distributedcomputing 2
    SADD tag:FP 2 3

さて、検索/選択の番です:

    a)  SINTER 'tag:erlang' 'tag:haskell'
        0 results

    b)  SINTER 'tag:programming' 'tag:computing'
        3 results: 1, 2, 3

    c)  SUNION 'tag:erlang' 'tag:haskell'
        2 results: 2 and 3

    d)  SDIFF 'tag:programming' 'tag:haskell'
        2 results: 1 and 2 (haskell is excluded)

### 検討 

SETs と ZSETs 操作は、検索する度に多くの結果をもたらす基本と応用の組み合わせられた構成要素です。
 
* (a) では、erlang 「かつ」 haskell にタグ付けされた本はありません。 
* (b) では、programming と computing にタグ付けされた「すべての本」を検索していることになります。
* erlang または haskell どちらかにタグ付けされた本は、 (c) で見つかります。
* (d) では、programming グループから haskell にタグ付けされたすべての本を除いています。

文書に基づいた保存と検索は、メタコードを 
<http://github.com/gleicon/docdb> で確認できます。そこでは過剰に見えるかもしれませんが、いくつかの文書化してみると、それほど多くの文書数にはなりません。

### 参照 

* <http://www.slideshare.net/gleicon/redis-3025589>
* <http://docs.python.org/library/sets.html>
* <http://en.wikipedia.org/wiki/Stemming>
* <http://tartarus.org/~martin/PorterStemmer/>
