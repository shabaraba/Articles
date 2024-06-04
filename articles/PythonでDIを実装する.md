# PythonでDIを実装する

Created: 2022年11月28日 17:02
Updated: 2022年12月1日 8:46
Published_Time: 2022年11月28日
Slug: di-with-python
Tags: DI, Python
Published: Yes
URL: https://blog.shaba.dev/posts/di-with-python

## はじめに

本業で使用する言語がJavaに代わり、DIの練習も兼ねてとある個人開発にDIを導入してみたくなりました。

現在pythonで実装していたので、[調べた結果](https://qiita.com/Jazuma/items/9fa15b36f61f9d1e770c)injectorというライブラリで実装すると綺麗にまとまりそうなので、その使い方と今の設計についてまとめてみます。

## DIとは

dependency injectionの略で、依存注入と訳されていますが、ただ言葉から意味を理解するのが凄い難しい単語だと思っています。
簡単に言えば、
`interfaceといろんなimplementを作って、プログラムを実行する際に使用したいクラスを指定して使う手法`
で、

具体的には

```python
import FugaService

class HogeController:
	def __init__(self) -> None:
		self.fuga_service = FugaService()

	def exec(self):
		self.fuga_service.piyo()
```

としていたのを

```python
import IFugaService

class HogeController:
	def __init__(self, fuga_service: IFugaService) -> None:
		self.fuga_service = fuga_service

	def exec(self):
		self.fuga_service.piyo()
```

と、内部で使用するクラスをコンストラクタの引数でもらってくるようにします（自分でFugaServiceをnewしない）。

その際、interfaceクラスを指定して受け取るようにして、DIコンテナと呼ばれるクラスにどのクラスを引数に渡すか決定してもらいます。
`依存関係を外からもらってくる`ので、「`依存注入`」なんですね。

### 何が嬉しいか

一番威力を発揮するのは`実行環境が変わる場合`でしょうか。

例えば`FugaService.piyo()`の戻り値が

- 開発環境ではメモリで保持していた値を返す
- 本番環境ではMySQLで保持していた値を取得して返す
- テスト実行時はMockで決め打ちの値を返す

のようにしたいニーズは、開発をスムーズに進める上で発生しうるかと思います。

このときの実装方針としては主に２つあって

1. `FugaService.piyo()`内で環境変数ごとにif文で出し分ける
2. `IFugaService`インターフェイスを継承した`DevFugaService`, `ProdFugaService`, `TestFugaService`クラスを作成して、`HogeController`のコンストラクタ内で環境変数ごとにif文でインスタンス化するクラスを決定する

になるかと思います。

ただ、`1.`の場合だとメソッド内の処理が複雑になりますし、`2.`の場合でもFugaServiceをインスタンス化する処理が他にもたくさんあったら修正して回るのがとても面倒ですよね。

DIを使用すれば、HogeControllerの外からFugaServiceが注入されるので、HogeControllerは実行環境について一切の知識が不要になります。
要は`コードがとてもきれいになる`のですね。
それはイイ。

## injector

injectorとはpythonでDIを実現するためのライブラリの一つで、使い方は上記でも貼った[Qiita記事](https://qiita.com/Jazuma/items/9fa15b36f61f9d1e770c)がわかりやすいと思います。

DIコンテナを作って、そこに依存関係を記載して、そのコンテナから注入したいクラスを呼ぶ感じで、とてもイメージどおりに書けます。

## サンプルと実例

今回のサンプルの役者と構成は以下の通り

実行すると「何かしらのメッセージをどこかしらに通知する」スクリプトを組むことにします。

```
src/
|`- infrastructures/
| |`- __init__.py
|  `- dependency_injector.py
|`- modules/
| `- notification/
|  |`- __init__.py
|   `- services/
|    |`- notification_service.py 
|     `- slack_notification_service_impl.py
|`- usecases/
| `- sample_usecase.py
`- main.py

```

module群（今回は一つだけですが）と、module全体で処理を共有するinfrastructuresモジュールで構成します。

modules内は通知モジュールが入っていて、今回はservices層だけが入っています。

infrastructures内は今回はDIコンテナだけが入っています。

### DIコンテナ

まずはDIコンテナから作成します。
インフラ層に置くことにします。

```python
import injector
from typing import Type, Any, Callable

class DependencyInjector:
	"""DIモジュール"""
	def __init__(self):
		# 依存性注入の初期化はconfigureに移譲
		self._injector = injector.Injector(self.__class__.configure)
		
	
	@classmethod
	def configure(cls, binder: injector.Binder) -> None:
		# コア部分で依存関係がわかっている場合はこの段階で書いちゃう
		pass
		
	
	def resolve(self, klass) -> Callable:
		# 与えられたインタフェースに応じて実体クラスを返す
		return self._injector.get(klass)
		
	
	def add(self, interface_, implement_):
		# 各モジュールで依存関係を設定したい場合に追加で登録する
		self._injector.binder.bind(interface_, to=implement_) # type: ignore[type-abstract]
	
dependency = DependencyInjector()
```

`dependency.add(A, B)`と書くことで、
Aクラスを引数に持つ場合は、Bクラスをインスタンス化して渡すことをinjectorに教えることができます。

`dependency.resolve(C)`

とすることで依存関係を解決した上でCクラスをインスタンス化できるようになります。

（詳しい例は後述）

### interfaceとimplement

まずは依存注入するためのクラスを作成します。
今回は通知モジュールのサービスクラスを対象にします。

要件では「どこかしらに通知する」としているので、一旦slackに通知するようにserviceをimplementして作成します。

```python
from abc import ABCMeta, abstractmethod

class INotificationService(metaclass=ABCMeta):
	def notify(self, message: str) -> bool:
		raise NotImplementedError
```

こちらがサービスのinterfaceクラス。
通知メソッドのみを持っています。
pythonはinterfaceという概念はないので、abcを用いた抽象クラスで代用します。
継承していなければエラーになるよう、NotImplementedErrorを投げるようにします。

続いてimplement側です。
pythonの細かい作法をわかっていないのですが、他言語に倣ってファイルを分けて作成しています。

```python
import slackweb

import infrastructures import SLACK_WEBHOOK_URL
import INotificationService

class SlackNotificationService(INotificationService):
	def notify(self, message: str) -> bool:
		try:
			slack = slackweb.Slack(url=SLACK_WEBHOOK_URL)
			slack.notify(text=message)
			return True
		except Exception as e:
			print(e)
			return False
```

別になんてことないですね。

implementは外から見える必要はないので、pythonの場合アンダースコアをつけてもいいかもですね。

### 依存関係の登録

では実際にDIコンテナにaddメソッドを用いて依存関係を登録します。

```python
"""init"""
from infrastructures import dependency
from .services.notification_service import NotificationService
from .services.slack_notification_service_impl import SlackNotificationServiceImpl

dependency.add(NotificationService, SlackNotificationServiceImpl)

__all__ = [
	"NotificationService",
]
```

作法については勉強不足でよくわかっていないのですが、個人的に各モジュール単位で依存関係を登録していったほうがわかりやすいんじゃないかなと思ったので、__init__.pyに記載してみています。

`dependency.add(NotificationService, SlackNotificationServiceImpl)`

としているので、

`NotificationService`を引数にもつ場合、`SlackNotificationServiceImpl`が代わりにインスタンス化されて渡されるようになります。

### 依存注入

お待ちかねの依存注入です。
今回はSampleUsecaseのコンストラクタに対して依存注入をしてみます。

```python
from injector import Inject
from modules.notification import NotificationService

class SampleUsecase:
	@Inject
	def __init__(self, notification: NotificationService) -> None:
		self.notification = notification
		self.__exec()

	def __exec(self) -> None:
		...
```

依存注入をする際は`@injector.Inject`アノテーションを使用します。
このアノテーションをつけることで、上述の通り`NotificationService`を引数にもつ場合、`SlackNotificationServiceImpl`が代わりにインスタンス化されて渡されるようになります。

これで、もしslack以外に通知するよう変更になった場合でも、
`XXXNotificationServiceImpl`を新たに作成して`modules.notification.__init__.py`の`dependency.add`の第2引数だけ修正を加えれば良くなります。

環境ごとに使用するサービスを変えるのも、`__init__.py`一箇所のみの対応でよくなります。

### 依存解決

では実際にSampleUsecaseに対して依存関係を解決してみます。

```python
from infrastructures import dependency
from usecases.sample_usecase import SampleUsecase

if __name__ == "__main__":
	dependency.resolve(SampleUsecase)
```

resolveメソッドを使用することで、SampleUsecaseをインスタンス化しようとする際に、`@Inject`アノテーションのあたっているコンストラクタの引数の依存関係を解決してくれます。

これを実行すると、晴れてslackでの通知処理が走るようになります。

## まとめ

javaだと当たり前なDIですが、pythonでも簡単に実装ができそうです。
現在絶賛練習中なので、なにか間違えていた場合は都度更新するようにします！