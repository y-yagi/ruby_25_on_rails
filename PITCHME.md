# Ruby 2.5 on Rails 5.2

### Ginza.rb / @y-yagi

---

## About

* Ruby 2.5に向けてしてRailsでも色々な対応が行われました(& 行われています). どのような対応が行われたか見てみましょう.
* なお、本資料は2017/12/17時点での情報で作成されています. 実際のコードと異なる可能性もあります. あしからず.

---

### [Remove the top-level `HashWithIndifferentAccess` contant](https://github.com/rails/rails/pull/27925)

* 下位互換性の為に残してあったtop levelの`HashWithIndifferentAccess`がdeprecateになった
* Ruby 2.5からトップレベルの定数は参照されなくなったのでついでに

---

## Conclusion

* 色んな人たちのおかげでRails 5.2はRuby 2.5で無事使えるようになっています. :pray:
* とはいえ、見落としもあると思うので、みんなでRuby 2.5 + Railsを 5.2使ってバグ踏んでいこうな
