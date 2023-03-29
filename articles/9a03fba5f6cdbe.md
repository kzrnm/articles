---
title: "【Python】argparse で「環境変数にあればそれを使う」を実現する方法"
emoji: "🌳"
type: "tech"
topics:
  - "python"
published: true
published_at: "2022-11-06 03:57"
---

## 前置き

「コマンドライン引数か環境変数で値を受け取る」という状況があります。

ここでは

```sh
python main.py --foo hoge --bar piyo
```

というようなコマンドを考えます。

- **foo**
  - コマンドラインで値を受け取る
  - コマンドラインがなく環境変数 `FOO` があればその値を使う
  - コマンドラインがなく環境変数 `FOO` もなければエラーとする
- **bar**
  - コマンドラインで値を受け取る
  - コマンドラインがなく環境変数 `BAR` があればその値を使う
  - コマンドラインがなく環境変数 `BAR` もなければ `defaultbar` とする


## 実装

### bar

`bar` の方は素直に実装すれば良さそうです。

```py
parser = argparse.ArgumentParser()
parser.add_argument("--bar", default=os.getenv("BAR", "defaultbar"))
```

### foo
foo の方は一工夫必要です。

```py
parser = argparse.ArgumentParser()
foo_value = os.getenv("FOO")
parser.add_argument("--foo", default=foo_value, required=not bool(foo_value))
```

`required` の値を環境変数によって変えるのが良いでしょう。

## より汎用的に

`add_argument` の `action` 引数に `argparse.Action` 型を入れるといろいろできるらしいです。
https://docs.python.org/ja/3.9/library/argparse.html#action

今回は環境変数を読み取るので動的に型を作ってあげると良いでしょう。

```py
import argparse
import os
from typing import Any, Optional, Sequence, Type, Union


def make_default_environ_action(envvar: str) -> Type[argparse.Action]:
    class DefaultEnvironAction(argparse.Action):
        def __init__(
            self, required: bool = False, default: Any = None, **kwargs: Any
        ) -> None:
            envvar_value = os.getenv(envvar)
            if envvar_value:
                required = False
                default = envvar_value
            else:
                required = True
                default = None
            super().__init__(default=default, required=required, **kwargs)

        def __call__(
            self,
            parser: argparse.ArgumentParser,
            namespace: argparse.Namespace,
            values: Union[str, Sequence[Any], None],
            option_string: Optional[str] = None,
        ) -> None:
            setattr(namespace, self.dest, values)

    return DefaultEnvironAction


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--foo", action=make_default_environ_action("FOO"), help="value or envvar $FOO")
    parser.add_argument("--bar", default=os.getenv("BAR", "defaultbar"))
    args = parser.parse_args()
    print(args)

```

## 結果

```sh
$ python3 main.py 
usage: util.py [-h] --foo FOO [--bar BAR]
util.py: error: the following arguments are required: --foo

$ python3 main.py --foo val
Namespace(bar='defaultbar', foo='val')

$ python3 main.py --foo val --bar var
Namespace(bar='var', foo='val')

$ FOO=VALUE python3 main.py
Namespace(bar='defaultbar', foo='VALUE')

$ FOO=VALUE BAR=VARIABLE python3 main.py
Namespace(bar='VARIABLE', foo='VALUE')

$ FOO=VALUE BAR=VARIABLE python3 main.py --foo aq --bar sw
Namespace(bar='sw', foo='aq')

$ python3 main.py -h
usage: util.py [-h] --foo FOO [--bar BAR]

optional arguments:
  -h, --help  show this help message and exit
  --foo FOO   value or envvar $FOO
  --bar BAR
```

というように環境変数の読み取りができるようになりました