---
marp: true
# dark theme
class: invert
---
<!-- headingDivider: 1 -->

# Rubyの組み込みメソッド 定義場所の確認方法

# Author

3akurur

# License

CC BY-NC-SA 4.0

[![CC BY-NC-SA 4.0](https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-nc-sa/4.0/)

Copyright (C) 2024 3akurur

# About this contents

<!-- GitHub Pages: [https://kaosf.github.io/20230803-git-tag](https://kaosf.github.io/20230803-git-tag)

Repository: [kaosf/20230803-git-tag - GitHub](https://github.com/kaosf/20230803-git-tag) -->

<!-- ![GitHub Pages QR](gh-pages-qr.svg) -->

# なぜ組み込みメソッドの定義箇所を探したくなったか

# なぜ組み込みメソッドの定義箇所を探したくなったか 2

緯度経度の情報が入った配列から、総走行距離を計算したい

```
logs = [[lat: 35.000, lng:135.000], [lat: 35.000, lng:135.000], [lat: 35.000, lng:135.000]]
```

# なぜ組み込みメソッドの定義箇所を探したくなったか 3

実際の実装は ```each_with_index``` を使って以下の様な感じで行いました。

```rb
array = Array.new(10) { rand(1..100) } # 実際は緯度経度情報
array.each_with_index do |value, index|
  next_value = array[index + 1]
  break if next_value.nil?
  p [value, next_value] # 実際は緯度経度の計算処理
end
```

# なぜ組み込みメソッドの定義箇所を探したくなったか 4

ただ、他の実装中にプロジェクト内で ```each_cons``` というメソッドを知り、以下の様に書き直せる事に気付きました。

```rb
array = Array.new(10) { rand(1..100) } # 実際は緯度経度情報
array.each_cons(2){|v| p v # 実際は緯度経度の計算処理
```

# なぜ組み込みメソッドの定義箇所を探したくなったか 5

each_consで書き直したけど、、、これって実際にはどう実装されてんの？
あわよくば ```each_with_index``` で書いたのと似たような感じで実装されてるとテンション上がるな。

# 実際に確認してみよう！

# 実際に確認してみよう！ 1

まずは以下のgemをinstall

```sh
gem install pry
gem install pry-doc
```

# 実際に確認してみよう！ 2

```
pry
require "pry-doc"
$ Enumerator#each_cons
```

実行すると次の様な出力が返ってきます

# 実際に確認してみよう！ 3

```
From: enum.c (C Method):
Owner: Enumerable
Visibility: public
Signature: each_cons(arg1)
Number of lines: 16

static VALUE
enum_each_cons(VALUE obj, VALUE n)
{
    long size = NUM2LONG(n);
    struct MEMO *memo;
    int arity;

    if (size <= 0) rb_raise(rb_eArgError, "invalid size");
    RETURN_SIZED_ENUMERATOR(obj, 1, &n, enum_each_cons_size);
    arity = rb_block_arity();
    if (enum_size_over_p(obj, size)) return obj;
    memo = MEMO_NEW(rb_ary_new2(size), dont_recycle_block_arg(arity), size);
    rb_block_call(obj, id_each, 0, 0, each_cons_i, (VALUE)memo);

    return obj;
}
```

# 実際に確認してみよう！ 5

上記は以下のサイトでも確認出来ます。

https://docs.ruby-lang.org/ja/latest/method/Enumerable/i/each_cons.html
https://docs.ruby-lang.org/en/3.3/Enumerable.html#method-i-each_cons
click to toggle source

# 実際に確認してみよう！ 4

ただ、他のメソッドも追っていっても何書いてるかよく分からない。。
~~引数が0以下だとerrorを返すとこだけわかる。~~ <-誰でも分かる

# 使用例を確認しよう

読んでも分からないので、使用例をコンソールで確認する方法を紹介します。

# 使用例を確認しよう 2

```
pry
require "pry-doc"
? Enumerator#each_cons
```

# 使用例を確認しよう 3

```
Calls the block with each successive overlapped n-tuple of elements;
returns self:

  a = []
  (1..5).each_cons(3) {|element| a.push(element) }
  a # => [[1, 2, 3], [2, 3, 4], [3, 4, 5]]

  a = []
  h = {foo: 0,  bar: 1, baz: 2, bam: 3}
  h.each_cons(2) {|element| a.push(element) }
  a # => [[[:foo, 0], [:bar, 1]], [[:bar, 1], [:baz, 2]], [[:baz, 2], [:bam, 3]]]

With no block given, returns an Enumerator.
```

※定義部分の下に表示されるが今回は長くなるので割愛

# 使用例を確認しよう 4

以下も同じ意味です。

```
# $ Enumerator#each_cons
$ show-source

# ? show-source -d
show-source -d Enumerator#each_cons
```

# 使用例を確認しよう 5

```
pry
ri Enumerator#each_cons
```

# 使用例を確認しよう 6

```
Enumerator#each_cons

(from ruby core)
Implementation from Enumerable
------------------------------------------------------------------------
  each_cons(n) { ... } ->  self
  each_cons(n)         ->  enumerator

------------------------------------------------------------------------

Calls the block with each successive overlapped n-tuple of
elements; returns self:

  a = []
  (1..5).each_cons(3) {|element| a.push(element) }
  a # => [[1, 2, 3], [2, 3, 4], [3, 4, 5]]

  a = []
  h = {foo: 0,  bar: 1, baz: 2, bam: 3}
  h.each_cons(2) {|element| a.push(element) }
  a # => [[[:foo, 0], [:bar, 1]], [[:bar, 1], [:baz, 2]], [[:baz, 2], [:bam, 3]]]

With no block given, returns an Enumerator.
```

# 余談：riは何の略か？

ググっても出ないのでダメ元でchatGPTに聞いてみると、 ```Ruby Interactive``` の略だそうです。
```Ruby Interactive``` でググっても何もヒットしなかったので、本当か尋ねると謝罪の後に「Ruby Index」の略とのこと。

こちらをググっても何も出てきませんでした。chatGPTにはご注意を。