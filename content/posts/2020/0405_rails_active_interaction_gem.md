---
title: "[Rails] ActiveInteractionについて"
date: 2020-04-05T23:16:07+09:00
toc: true
tags: [
  "Gem",
  "Ruby",
  "Rails",
  "デザインパターン"
]
---

最近業務でよく使う[ActiveInteraction](https://github.com/AaronLasseigne/active_interaction)というGemについて、利用する目的を落とし込みたく、このGemを使うことでどのようなメリットがあるのかを深堀りしようと記事にすることにしました。

GitHubリポジトリにはこのように書いてあります。

> ActiveInteraction manages application-specific business logic. It's an implementation of the command pattern in Ruby.

どうやら[デザインパターンの一種であるcommandパターン](https://www.techscore.com/tech/DesignPattern/Command.html/)が使われているようです。

また、このCommandパターンについては、

> Commandパターンでは、ある特定の動作を実行するオブジェクトを構築します。「特定」という部分が重要です。Commandパターンのコマンドのインスタンスはどの社員の住所についても変更する方法を知っているわけではありません。代わりに、ある特定の社員を新しい住所に移す方法を知っています。

> コマンドは何かを行う方法を知っているだけで、それを実行する対象の状態には関心がありません。

参考：Rubyによるデザインパターン

この表現がわかりやすかったです。
特定のビジネスロジックをコマンドとして作成し、通常の実行メソッドと併せて元に戻す（アンドゥ）メソッドも用意することができます。ActiveRecordのマイグレーション機能はCommandパターンの実装の典型例です。

話が脱線しましたが、以上のCommandパターンをActiveInteractionクラスを継承することで実現できるようです。

ActiveInteractionを使うことで得られるメリットについて考えていきます。

## 1. Fat Controllerになることを未然に防ぐことができる
コントローラは複雑になりがちであり、コード量が段々と膨れ上がる傾向があります。
```ruby
def create
  service = CreateUserService.run(user_params)

  if service.valid?
    head :created
  else
    render service.errors, status: :bad_request
  end
end
```
ActiveInteractionは、コントローラとは別にビジネスロジックを記述しておく場所を提供してくれます。
ActiveInteractionクラスを継承したServiceクラスに特定のビジネスロジックを書いておけば、コントローラでは`run`メソッドで呼び出すだけで済むようになるので、簡潔になりますね。(リファレンスでは `app/interactions/` 配下に配置することを推奨しています)

## 2. ロジックを分離できるため、テストが書きやすくなる
これは1つ目のメリットに繋がるものではありますが、ロジックに集中したテストが書けるのでテスト単位が小さくなるのでテストが書きやすくなります。
業務ではサービスクラスのテスト、コントローラのテストを分けておりますが、2つに分けることでテストも簡潔になりますし、どういったコードの動きを担保したいのかがよりわかりやすくなるので、コントローラと同様に保守性の高いテストコードを保つことができます。


## 3. より安全なコードを保つことができる
ActiveInteractionは型安全性を提供しているので、型の不一致によるバグが起こりにくく安全なコードを保つことができます。また、使用頻度は高くありませんが、ActiveModelと同様の書き方でバリデーションを定義することもできます。

```ruby
class Users::SignUpService < ActiveInteraction::Base
  string :name, :email, password

  def execute
    user = User.save(inputs)
    unless user
      errors.add(:message, "値が不正です")
    end
  end
end
```

上記のコードでは、inputsで受け入れる3つのStringパラメータを定義しています。

個人的に型の恩恵が受けられるのは、バグを発生させにくいコードを書くことに繋がると思っているので非常に大きなメリットだと思っています。型の種類がなかなか豊富で柔軟にロジックが記述できそうなのもいいですね。


## 最後に
ActiveInteractionを使ってロジックを分離して書くことで、保守性や再利用性の高いコードが実現できそうですね。
[ActiveInteractionのREADME](https://github.com/AaronLasseigne/active_interaction)は非常に充実しており、基本的な使い方から発展的な使い方まで書いてあるので、使い方に迷った際には迷わずREADMEを参照でよさそうです。
