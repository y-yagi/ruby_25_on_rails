# Ruby 2.5 on Rails 5.2

### Ginza.rb / @y-yagi

---

## About

* Ruby 2.5に向けてしてRailsでも色々な対応が行われました(& 行われています). どのような対応が行われたか見てみましょう.
* なお、本資料は2017/12/17時点での情報で作成されています. 実際のコードと異なる可能性もあります. あしからず.

---

### [Hash#transform_keys is in Ruby 2.5+](https://github.com/rails/rails/commit/f213e926892020f9ab6c8974612c59e2ba959253)

* Ruby本体に`Hash#transform_keys`が追加されたので、`transform_keys`が既に定義済みなら、そちらを使用するよう対応

---

### [Let Hash#slice return a Hash](https://github.com/rails/rails/commit/01ae39660243bc5f0a986e20f9c9bff312b1b5f8)

* Ruby本体に`Hash#slice`が追加されたので、`slice`が既に定義済みなら、そちらを使用するよう対応
* 合わせて、`Hash#slice`メソッドが必ず`Hash`クラスのインスタンスを返すよう修正
  * Rubyに組み込みの`Hash#slice`と挙動を合わせる為

---

### [Let Hash#slice return a Hash](https://github.com/rails/rails/commit/01ae39660243bc5f0a986e20f9c9bff312b1b5f8)

* 元々は`self.class.new`を返すようになっていたので、`Hash`を拡張したクラスで`slice`を使用した場合の結果が変わるので注意が必要
  * `HashWithIndifferentAccess`については変わらず`HashWithIndifferentAccess`のインスタンスを返すよう対応済み

---

### [Remove the top-level `HashWithIndifferentAccess` contant](https://github.com/rails/rails/pull/27925)

* 下位互換性の為に残してあったtop levelの`HashWithIndifferentAccess`がdeprecateになった
* Ruby 2.5からトップレベルの定数は参照されなくなったのでついでに

---

### [Fix duplicable? for Rational and Complex](https://github.com/rails/rails/commit/7045e03dee2cda9af65bc8e51bddab79599f44cd)

* Rational、ComplexがRuby 2.5ではdup出来るようになった為、それに合わせて`Complex#duplicable?`、`Rational#duplicable?`メソッドがtrueを返すようになった

---

### [double assign is no longer an effective workaround for unused variable warning](https://github.com/rails/rails/commit/61f92f8bc5fa0b486fc56f249fa23f1102e79759)

* Rubyのwanring(assigned but unused variable)を避ける為にダブルアサインを使用する、というテクニックがあった
* が、Ruby 2.5では上記テクニックが使えなくなった(普通にwarningが出るようになった)ので、自身に代入する、という対応を行った

---

## ここからテストの修正のみ

---

### [Address LogSubscriberTest failures to support Rails 2.5.0-dev](https://github.com/rails/rails/pull/29089)

* Integerのround/floor等の数値を丸めるメソッドに、引数で小数点以下の桁数を指定した時にも Integer を返すようにする仕様変更が入った
  * Integer を Float にした時に精度が足りずに値が変化してしまうのを避けるためだそうな
  * [Feature \#13420: Integer\#\{round,floor,ceil,truncate\} should always return an integer, not a float](https://bugs.ruby-lang.org/issues/13420)

---

### [ERB::Util.url_encode no longer escapes ~ since ruby 2.5](https://github.com/rails/rails/commit/f8b5b4af843cb3107071c7d9fdc0d76bb43c47d6)

* `ERB::Util.url_encode`メソッドが`~`をエスケープしなくなった
  * [Bug \#6696: \[PATCH\] ERB::Util\.url\_encode should not escape unreserved characters](https://bugs.ruby-lang.org/issues/6696)
* エスケープされなくなったのは、`~`はUnreserved Charactersに含まれていなかった為
  [RFC 3986 \- Uniform Resource Identifier \(URI\): Generic Syntax](https://tools.ietf.org/html/rfc3986#section-2.3)

---

### [Suppress `warning: BigDecimal.new is deprecated`](https://github.com/rails/rails/commit/e4a6a23aa77185127ce9609777820fab14a689bb)

* BigDecimal 1.3.3で`BigDecimal.new`がdeprecatedになり、`BigDecimal`のインスタンスを生成するには`Kernel.BigDecimal`を使わなければならなくなった
  * BigDecimalをnumericクラスのようにimmutable + frozenにする為らしい
  * [Removing BigDecimal\.new to match core numeric classes like Integer](https://github.com/ruby/bigdecimal/issues/89)

---

### [Suppress expected exceptions by `report_on_exception` = `false` in Ru…](https://github.com/rails/rails/commit/1d3fe75649e9e5dd9efacb7a6a0d9e9d12b3df34)

* `Thread#report_on_exception`のデフォルトがtrueになった事により本来不要なエラーも表示されるようになった
  * 不要な箇所では`report_on_exception`に`false`を指定するようにして対応した
* が、これにより気付けたエラーもあったので、良かった

---

## 惜しくも入らなかった対応

---

### [Using require_relative in the Rails codebase](https://github.com/rails/rails/pull/29638)

* 各コンポーネントで`require`の代わりに`require_relative`を使用するよう対応
* `require_relative`は、`require`よりもはやい(`$LOAD_PATH`をスキャンせず対象のファイルを直接読み込む為)為
  * また、`require_relative`は`require`と異なり、RubyGemsやBundlerにオーバーライドされてない為、不要なパッチを避ける事が出来る、というのもあるらしい

---

### [Using require_relative in the Rails codebase](https://github.com/rails/rails/pull/29638)

* が、Ruby 2.4までだと`require_relative`はシンボリックリンク先のファイルをロード出来ない(Rubyのインストール先がシンボリックリンクを使用している場合ファイルをロード出来ずエラーになる)、という問題があった為、後ほどrevert
  * Ruby 2.5では上記問題は対応済み
* Rails 6.0でどうだろう

---

## Conclusion

* 色んな人たちのおかげでRails 5.2はRuby 2.5で無事使えるようになっています. :pray:
* とはいえ、見落としもあると思うので、みんなでRuby 2.5 + Railsを 5.2使ってバグ踏んでいこうな
