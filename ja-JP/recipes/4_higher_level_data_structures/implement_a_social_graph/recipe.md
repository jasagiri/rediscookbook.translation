### 問題

ある種のアプリケーションでユーザのソーシャルグラフを実装するのに単一または複数の方向の関係が利用可能な状態（ following や friendship ）で Redis を使いたいとしましょう。

### 解法 

Redis 組み込みの機能豊富な set を使って、各ユーザのユニークID をキーとした「follow」、「follower」と「blocked」リストを構成します。redis の生データではこのように見えます：

    redis> SADD user:1:follows 2
    (integer) 1
    redis> SADD user:2:followers 1
    (integer) 1
    redis> SADD user:3:follows 1
    (integer) 1
    redis> SADD user:1:followers 3
    (integer) 1
    redis> SADD user:1:follows 3
    (integer) 1
    redis> SADD user:3:followers 1
    (integer) 1
    redis> SINTER user:1:follows user:1:followers
    1. 3
    
### 検討 

Redis はキーに割り当てられたユニークな値の集まりの「sets」を構成することができます。
特定のユーザ用に「follows」「followers」リストの両方を作成し、簡単な積集合を計算することで彼らの「friendships」を求められるでしょう。

Ruby でそのシステムを実装する場合には、このような感じです。：

{% code_snippet social_graph.rb %}

### 参照
