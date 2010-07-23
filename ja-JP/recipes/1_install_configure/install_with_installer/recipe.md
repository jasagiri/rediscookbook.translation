### 問題

以下のことをしたい：

* Linux や OS X や Posix 互換のオペレーティングシステムに Redis をインストールする。
* 標準レイアウトを使った設定でバイナリを配置する。
* root にインストールしたい場合は /etc/{init.d,rc.d}/redis にシステムコントロールスクリプトがある。
* １つ以上のリモートホストにインストールする。
* *１つ* のコマンドを使って、ローカルにインストール可能であること。

### 解法

[redis-installer](http://github.com/wayneeseguin/redis-installer/) を使ってください。

1. localhost にインストールします：

まずは、redis-installer をダウンロードまたはクローンします。: 

	git clone git://github.com/wayneeseguin/redis-installer/

次に、単に Redis をインストールします。：

	bin/install-redis

2. **１つ** のコマンドで localhost にインストールします。

"怠けてるのではなく、賢いんです！"

	bash < <(curl http://github.com/wayneeseguin/redis-installer/raw/master/bin/install-redis)

3. 複数のリモートホストにインストールします

まず、下記から redis-installer をダウンロードまたはクローンします。: 

	git clone git://github.com/wayneeseguin/redis-installer/

次に、１つ以上のリモートホストに Redis をインストールします。

	bin/install-redis-on-hosts hostname1 [hostname2 [hostname3 ...]]

### 検討

インストールと設定で気をつけることは、 root としてインストールするか user としてするかで異なるということです。 root では、/usr/local/ に、 user では~/.redis/ にインストールされます。

解法２は単に Redis インストーラスクリプトをダウンロードして、`bash` に直接実行コマンドを送んでいるだけです。

### 参照
