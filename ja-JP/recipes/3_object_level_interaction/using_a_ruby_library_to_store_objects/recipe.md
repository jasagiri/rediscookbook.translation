### 問題

Ruby オブジェクトを永続化する [remodel](http://github.com/tlossen/remodel) を使いたい。

### 解法

オプションを探し、1つ選んでください！ Ruby には オブジェクトを保存するいくつかの永続化ライブラリがあります続化オブジェクトが存在しています。
このレシピでは、例として [Ohm](http://github.com/soveran/ohm) と [remodel](http://github.com/tlossen/remodel) を取り上げます。
しかし、読者はこの目的のために使用できる他の良いライブラリがあることを忘れないでください。
[どうか、他のライブラリやたくさんの使用例を追加してください。]

一般的な抽象化戦略は複雑なスキーマ定義無しでオブジェクトをマッピングすることで、
同等の結果を得るためにとても簡単なドメイン固有言語を使用しています。

次に Ohm の例を考えてください。
ここでは、３つの任意のオブジェクト（Event, Venue, および Person）をモデル化し、Event にバリデーションを提供するつもりです。

	class Event < Ohm::Model
  	  attribute :name
  	  reference :venue, Venue
  	  set :participants, Person
  	  counter :votes

 	  index :name

   	  def validate
   	    assert_present :name
	  end
	end

	class Venue < Ohm::Model
  	  attribute :name
  	  collection :events, Event
	end

	class Person < Ohm::Model
 	 attribute :name
	end
 
Remodel は永続化エントリの説明的なシンプルな DSL を提供します。：
よく知られている 'has_many' 関連は remodel ではこうです。

	require 'remodel'
	
	class Cookbook < remodel::Entity
	  has_many :recipes, :class => 'Recipe', :reverse => :book
	  property :title, :class => String
	  property :author, :class => String
	end
	
	class Recipe < remodel::Entity
	  has_one :book, :class => Cookbook, :reverse => :recipes
	  property :name, :class => String
	end

もし上記を `cookbook.rb` として保存して、Redis をローカルで動かせるなら、
Ruby シェルを開いて、実行できます。

	>> require 'cookbook.rb'
	=> true
	>> book = Cookbook.create :title => 'Python Cookbook', :author => 'Alex Martelli'
	=> #<Cookbook(c:1) title: "Python Cookbook", author: "Alex Martelli">
	>> recipe = book.recipes.create :name => 'Sorting a Dictionary'
	=> #<Recipe(r:4) name: "Sorting a Dictionary">
	>> recipe.book
	=> #<Cookbook(c:1) title: "Python Cookbook", author: "Alex Martelli">

### 検討

永続化オブジェクトのために既存のライブラリを使う重要な理由は、
組み込みのbuilt-in *アサーション* や バリデーションの力です。
例えば Ohm では、数値のみフィールドには確かに数値にマッチするかをアサーションします。
`format` バリデーションのために正規表現を使うことが出来ます。:

	assert_format :username, /^\w+$/

Remodel はそれ自身「最小のオブジェクト-redis マッパー」として定義され、簡単なマッピング戦略に使います。：
オブジェクトのすべてのプロパティは JSON ハッシュにシリアライズされ、単一キー配下に保存されます。

関連の扱いは異なっており、専ら &mdash; で扱われます。
関連の両端は別々のキーに保存されます。
`has_many` は Redis list を使い、関連オブジェクトのキーを保存し、
`has_one` は Redis string を使い単一の関連キーを保存します。
この手法は、どんなオブジェクトでも（デ）シリアライズすること無く、関連を変更できるという利点があります。

### 参照
