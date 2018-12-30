文字列内の立っているビット数（population count）を数えます。

By default all the bytes contained in the string are examined.
It is possible to specify the counting operation only in an interval passing the
additional arguments *start* and *end*.

Like for the `GETRANGE` command start and end can contain negative values in
order to index bytes starting from the end of the string, where -1 is the last
byte, -2 is the penultimate, and so forth.

Non-existent keys are treated as empty strings, so the command will return zero.

@return

@integer-reply

The number of bits set to 1.

@examples

```cli
SET mykey "foobar"
BITCOUNT mykey
BITCOUNT mykey 0 0
BITCOUNT mykey 1 1
```

## パターン：ビットマップを使用したリアルタイムメトリクス

Bitmaps are a very space-efficient representation of certain kinds of
information.
One example is a Web application that needs the history of user visits, so that
for instance it is possible to determine what users are good targets of beta
features.

Using the `SETBIT` command this is trivial to accomplish, identifying every day
with a small progressive integer.
For instance day 0 is the first day the application was put online, day 1 the
next day, and so forth.

Every time a user performs a page view, the application can register that in
the current day the user visited the web site using the `SETBIT` command setting
the bit corresponding to the current day.

Later it will be trivial to know the number of single days the user visited the
web site simply calling the `BITCOUNT` command against the bitmap.

日数の代わりにユーザーIDが使用される同様のパターンは、 "[Redisビットマップを使用した高速で簡単なリアルタイムメトリクス](http://blog.getspool.com/2011/11/29/fast-easy-realtime-metrics-using-redis-bitmaps) "という記事で説明されています 。

## Performance considerations

上記の日数計算の例では、10年経ってもアプリケーションがオンラインの場合でも、1ユーザーあたり`365*10`ビットのデータ、つまり1ユーザーあたりわずか456バイトしかありません。このデータ量では、 `BITCOUNT`や`GET`や`INCR` ような他の O(1) Redisコマンドと同じくらい速いです。

When the bitmap is big, there are two alternatives:

- Taking a separated key that is incremented every time the bitmap is modified.This can be very efficient and atomic using a small Redis Lua script.
- Running the bitmap incrementally using the `BITCOUNT` *start* and *end*optional parameters, accumulating the results client-side, and optionallycaching the result into a key.
