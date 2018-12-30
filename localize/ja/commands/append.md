`key`がすでに存在し、文字列である場合、このコマンドは文字列の末尾に`value`を追加します 。 keyが存在しない場合は作成され、空の文字列として設定されます。したがって、 `APPEND` はこの特別な場合の`SET`に似ています。

@return

@integer-reply: the length of the string after the append operation.

@examples

```cli
EXISTS mykey
APPEND mykey "Hello"
APPEND mykey " World"
GET mykey
```

## パターン: 時系列

The `APPEND` command can be used to create a very compact representation of a
list of fixed-size samples, usually referred as *time series*.
Every time a new sample arrives we can store it using the command

```
APPEND timeseries "fixed-size sample"
```

Accessing individual elements in the time series is not hard:

- `STRLEN` can be used in order to obtain the number of samples.
- `GETRANGE` allows for random access of elements.If our time series have associated time information we can easily implementa binary search to get range combining `GETRANGE` with the Lua scriptingengine available in Redis 2.6.
- `SETRANGE` can be used to overwrite an existing time series.

The limitation of this pattern is that we are forced into an append-only mode
of operation, there is no way to cut the time series to a given size easily
because Redis currently lacks a command able to trim string objects.
However the space efficiency of time series stored in this way is remarkable.

ヒント: 現在のUnix時間に基づいて別のキーに切り替えることができます。このようにすると、キーあたりのサンプル数を比較的少なくして、非常に大きなキーを扱わなくてもよくなり、このパターンを多くのRedisインスタンスに分散させるのが簡単になります。

固定サイズの文字列を使用してセンサーの温度をサンプリングする例（実際にはバイナリ形式を使用して実装すると良いでしょう）。

```cli
APPEND ts "0043"
APPEND ts "0035"
GETRANGE ts 0 3
GETRANGE ts 4 7
```
