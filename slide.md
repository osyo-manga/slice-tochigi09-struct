#### とちぎRuby会議09
- - -

## 今更聞けない！
## Struct の使い方と今後の可能性について



---

#### 自己紹介
- - -

* 名前：osyo
* Twitter : [@pink_bangbi](https://twitter.com/pink_bangbi)
* github  : [osyo-manga](https://github.com/osyo-manga)
* ブログ  : [Secret Garden(Instrumental)](http://secret-garden.hatenablog.com)
* Rails 歴2年半
* 趣味で Ruby にパッチを投げたり bugs.ruby で気になったチケットをブログにまとめたりしてる
* Ruby で一番好きな機能は Refinements

---

## Struct の使い方と
## 今後の可能性について

---

# Struct とは！

---


#### Struct とは
- - -

* 任意のプロパティを持ったクラスを動的に生成する Ruby の標準ライブラリ               <!-- .element: class="fragment" -->

```ruby
# 3つのプロパティを持つ疑似クラスを作成する
User = Struct.new(:id, :name, :age)

# 作成した疑似クラスは通常のクラスと同じように使用できる
# 引数は Struct.new に渡した順で割り当てられる
homu = User.new(1, "homu", 14)


# 各プロパティのアクセッサが暗黙的に定義されている
p homu.name  # => "homu"
p homu.age   # => 14
homu.age = 15
p homu.age    # => 15
```
<!-- .element: class="fragment" -->

---

#### 参照方法いろいろ
- - -

```ruby
# アクセッサメソッドとして参照できる
p homu.name    # => "homu"
p homu.age     # => 14

# [] メソッドで参照
# Hash みたいにキーを渡してアクセス
p homu[:name]   # => "homu"

# index を渡してアクセス
p homu[1]       # => 15

# Hash に変換できる
p homu.to_h
# => {:id=>1, :name=>"homu", :age=>14}

# 未定義のプロパティにはアクセスできない
p homu.hoge    # error: undefined method `hoge'
p homu[:foo]   # error: `[]': no member 'foo' in struct (NameError)
```

---

#### 定義方法いろいろ
- - -

```ruby
# 3つのプロパティを持つ疑似クラスを作成する
User = Struct.new(:id, :name, :age)
# User.new には Struct.new で渡した引数の順番で渡す
User.new(1, "homu", 14)
```

```ruby
# Struct.new に keyword_init: true を渡すと
User = Struct.new(:id, :name, :age, keyword_init: true)
# User.new にキーワード引数で渡せるようになる
p User.new(name: "homu", age: 14, id: 1)
# => #<struct User id=1, name="homu", age=14>
```

```ruby
# Struct.new にブロックを渡し、その中でメソッドを定義すると
# インスタンスメソッドとして定義される
User = Struct.new(:last_name, :first_name) do
  def full_name
    "#{last_name} #{first_name}"
  end
end
homu = User.new("巴", "マミ")
# ユーザが定義したインスタンスメソッドが呼べる
p homu.full_name    # => "巴 マミ"
```

---

### Struct ってどういう時に使うの？

---

#### Struct を継承する
- - -

```ruby
class User < Struct.new(:last_name, :first_name, :age)
  def full_name
    "#{last_name} #{first_name}"
  end
  def to_s
    "#{full_name} #{age}歳"
  end
end

homu = User.new("巴", "マミ", 15)
# Struct のメソッドがそのまま使える
p homu.last_name   # => "巴"
p homu.to_h        # => {:last_name=>"巴", :first_name=>"マミ", :age=>15}
```

* Struct.new はクラスオブジェクトを返す                 <!-- .element: class="fragment" -->
* なので Struct.new を継承することができる          <!-- .element: class="fragment" -->
* Struct.new を継承することで Struct の便利メソッドがそのまま使える！！          <!-- .element: class="fragment" -->

---

#### 引数や戻り値の擬似オブジェクトとして使う
- - -

```ruby
def show_file(file)
  puts "path: #{file.path}"
  puts "size: #{file.size}"
  puts "lines:"
  puts file.readlines
end
```
<!-- .element: class="fragment" -->
```ruby
# 普通はファイルを渡して使う
show_file File.open("./test.rb")
```
<!-- .element: class="fragment" -->

```ruby
# Tempfile というクラスをその場で定義してそのインスタンスを渡す
Tempfile = Struct.new(:path, :size, :readlines)
show_file Tempfile.new("./test.rb", 14, %w(homu mami mado))
```
<!-- .element: class="fragment" -->

* その場でデータ構造を定義してオブジェクトを生成する事ができる       <!-- .element: class="fragment" -->
* テストとかの mock オブジェクトとかでも利用できる   <!-- .element: class="fragment" -->
    * 疑似ファイルをテスト上で定義したりとか   <!-- .element: class="fragment" -->
    * API モジュールの戻り値として定義したりとか   <!-- .element: class="fragment" -->

---

## Struct の今後の可能性…？

---

#### [[Feature #16986] Anonymous Struct literal](https://bugs.ruby-lang.org/issues/16986)
- - -

* Anonymous Struct literal という機能が提案されている         <!-- .element: class="fragment" -->
    * ${} というリテラルで Struct のオブジェクトを定義できるようにする提案
    * これを使うと Hash みたいにカジュアルに Struct オブジェクトが使える

```ruby
# 今の書き方
Struct.new(:a, :b).new(1, 2)
```
<!-- .element: class="fragment" -->

```ruby
# 提案してるリテラルだと ${} で定義できる
${ a: 1, b: 2 }
```
<!-- .element: class="fragment" -->

```ruby
# さっきのコード例
Tempfile = Struct.new(:path, :size, :readlines)
show_file Tempfile.new("./test.rb", 14, %w(homu mami mado))
```
<!-- .element: class="fragment" -->

```ruby
# ${} ですっきりとかける
show_file ${ path: "./test.rb", size: 14, readlines: %w(homu mami mado) }
```
<!-- .element: class="fragment" -->

---

#### Hash との比較例
- - -

```ruby
# Hash
homu = { id: 1, name: "homu", age: 14 }

# [] でのみ要素にアクセスできる
homu[:name]
# 新しい要素を追加できる
homu[:job] = "魔法少女"
# 存在しないキーにアクセスしてもエラーにならない
homu[:nmae]  # => nil
```
<!-- .element: class="fragment" -->

```ruby
# Anonymous Struct literal
homu = ${ id: 1, name: "homu", age: 14 }

# [] だけでなくて . で参照できる
homu.name

# 新しい要素は追加できない
# そもそも存在しない要素にアクセスするとエラーになる
homu[:job] = "魔法少女"    # error: undefined method `[]'
```
<!-- .element: class="fragment" -->

---

## まだ議論中です！

---


#### 今どうなってる？
- - -

* 入るかどうかすら決まってません！！
* いろんな書き方の提案がされている
    * https://bugs.ruby-lang.org/issues/16986#note-11
    * <del>${} の $ は Struct の S です</del>

```ruby
${a:1, b:2}        # 元々の提案
{|a:1, b:2|}       # <- matz のアイデア
struct a: 1, b: 2  # struct キーワードを追加
%o{a:1, b:2}       # %記法で定義
(a:1, b:2)         # {} ではなくて () で定義
Struct.anonymous(a:1, b:2)   # メソッド定義
Struct(a:1, b:2)
Struct[a:1, b:2]
```

* そういえば Rubykaigi で matz が新しいシンタックスを入れたくないって言っていたような…？    <!-- .element: class="fragment" -->
---

## まとめ

---

#### まとめ
- - -

* Struct はその場でちょっとしたデータ構造を持つオブジェクトを定義する時に便利         <!-- .element: class="fragment" -->
* 継承して使うとそのクラスのプロパティ周りの処理がスッキリする         <!-- .element: class="fragment" -->
* mock やダックタイピングで呼び出されるオブジェクトを定義する時とかにも利用できる         <!-- .element: class="fragment" -->
* Anonymous Struct literal ほしい！！！         <!-- .element: class="fragment" -->
* ${} みたいな機能は実装されてからみんな使い始めて便利、みたいになりそう         <!-- .element: class="fragment" -->


---

# 宣伝

---

### 09/19 に [ROM専歓迎 | RubyKaigi Takeout 2020 感想戦＠仮想松本](https://smarthr.connpass.com/event/187270/) やります！
### 気になる人がいればぜひぜひ参加を！

---

## ご清聴
## ありがとうございました

