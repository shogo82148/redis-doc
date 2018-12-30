このコマンドは、Redis文字列をビットの配列として扱い、さまざまなビット幅および任意の（必要な）位置合わせされていないオフセットの特定の整数フィールドをアドレス指定できます。実際には、このコマンドを使用して、たとえばビットオフセット1234の符号付き5ビット整数を特定の値に設定し、オフセット4567から31ビットの符号なし整数を取得することができます。ユーザーが設定できる、保証され、明確に指定されたオーバーフローおよびアンダーフローの動作。

`BITFIELD`は、同じコマンド呼び出しで複数のビットフィールドを操作することができます。実行する操作のリストを受け取り、応答の配列を返します。各配列は引数のリスト内の対応する操作と一致します。

たとえば、次のコマンドはビットオフセット100で8ビット符号付き整数をインクリメントし、ビットオフセット0で4ビット符号なし整数の値を取得します。

```
> BITFIELD mykey INCRBY i5 100 1 GET u4 0
1) (integer) 1
2) (integer) 0
```

Note that:

1. Addressing with `GET` bits outside the current string length (including the case the key does not exist at all), results in the operation to be performed like the missing part all consists of bits set to 0.
2. Addressing with `SET` or `INCRBY` bits outside the current string length will enlarge the string, zero-padding it, as needed, for the minimal length needed, according to the most far bit touched.

## サポートされているサブコマンドと整数型

The following is the list of supported commands.

- **GET** `<type>` `<offset>` - 指定されたビットフィールドを返します。
- **SET** `<type>` `<offset>` `<value>` - 指定されたビットフィールドを設定し、その古い値を返します。
- **INCRBY** `<type>` `<offset>` `<increment>` - 指定したビットフィールドを増分または減分し（負の増分が指定されている場合）、新しい値を返します。

オーバーフロー動作を設定することによって、連続した`INCRBY`サブコマンド呼び出しの動作を変更するだけのサブコマンドもあります。

- **OVERFLOW** `[WRAP|SAT|FAIL]`

Where an integer type is expected, it can be composed by prefixing with `i` for signed integers and `u` for unsigned integers with the number of bits of our integer type. So for example `u8` is an unsigned integer of 8 bits and `i16` is a
signed integer of 16 bits.

サポートされている型は、符号付き整数の場合は最大64ビット、符号なし整数の場合は最大63ビットです。符号なし整数に関するこの制限は、現在Redisプロトコルが応答として64ビット符号なし整数を返すことができないという事実によるものです。

## ビットと位置オフセット

bitfieldコマンドでオフセットを指定するには2つの方法があります。接頭辞を付けずに数値を指定すると、文字列内のゼロベースのビットオフセットとして使用されます。

However if the offset is prefixed with a `#` character, the specified offset
is multiplied by the integer type width, so for example:

```
BITFIELD mystring SET i8 #0 100 i8 #1 200
```

最初のi8整数をオフセット0に設定し、2番目の整数をオフセット8に設定します。このようにして、必要なものが特定のサイズの整数の単純な配列であれば、自分で計算を行う必要はありません。

## オーバーフロー制御

`OVERFLOW`コマンドを使用して、ユーザーは以下の動作のいずれかを指定することによって、増分または減分のオーバーフロー（またはアンダーフロー）の動作を微調整できます。

- **WRAP**: wrap around, both with signed and unsigned integers. In the case of unsigned integers, wrapping is like performing the operation modulo the maximum value the integer can contain (the C standard behavior). With signed integers instead wrapping means that overflows restart towards the most negative value and underflows towards the most positive ones, so for example if an `i8` integer is set to the value 127, incrementing it by 1 will yield `-128`.
- **SAT**: uses saturation arithmetic, that is, on underflows the value is set to the minimum integer value, and on overflows to the maximum integer value. For example incrementing an `i8` integer starting from value 120 with an increment of 10, will result into the value 127, and further increments will always keep the value at 127. The same happens on underflows, but towards the value is blocked at the most negative value.
- **FAIL**: in this mode no operation is performed on overflows or underflows detected. The corresponding return value is set to NULL to signal the condition to the caller.

各`OVERFLOW`ステートメントは、次の`OVERFLOW`ステートメントまで、サブコマンドのリスト内でそれに続く`INCRBY`コマンドにのみ影響することに注意してください。

特に指定がなければ、デフォルトで**WRAP**が使用されます。

```
> BITFIELD mykey incrby u2 100 1 OVERFLOW SAT incrby u2 102 1
1) (integer) 1
2) (integer) 1
> BITFIELD mykey incrby u2 100 1 OVERFLOW SAT incrby u2 102 1
1) (integer) 2
2) (integer) 2
> BITFIELD mykey incrby u2 100 1 OVERFLOW SAT incrby u2 102 1
1) (integer) 3
2) (integer) 3
> BITFIELD mykey incrby u2 100 1 OVERFLOW SAT incrby u2 102 1
1) (integer) 0
2) (integer) 3
```

## 戻り値

The command returns an array with each entry being the corresponding result of
the sub command given at the same position. `OVERFLOW` subcommands don't count
as generating a reply.

以下は、NULLを返す`OVERFLOW FAIL`例です。

```
> BITFIELD mykey OVERFLOW FAIL incrby u2 102 1
1) (nil)
```

## 動機

このコマンドの動機は、多数の小さい整数を1つの大きなビットマップとして格納する（または巨大なキーを使用しないようにいくつかのキーに分割する）ことはメモリ効率が非常に高く、特にRedisの新しいユースケースが適用されることです。リアルタイム分析の分野。このユースケースは、制御された方法でオーバーフローを指定する機能によってサポートされています。

Fun fact: Reddit's 2017 April fools' project [r/place](https://reddit.com/r/place) was [built using the Redis BITFIELD command](https://redditblog.com/2017/04/13/how-we-built-rplace/) in order to take an in-memory representation of the collaborative canvas.

## パフォーマンスの考慮事項

Usually `BITFIELD` is a fast command, however note that addressing far bits of currently short strings will trigger an allocation that may be more costly than executing the command on bits already existing.

## ビットの順序

`BITFIELD`使用される`BITFIELD`は、ビットマップ0を最初のバイトの最上位ビットと見なし、以下同様にオフセット7の値23に5ビット符号なし整数を設定するビットマップを考慮します。すべてゼロの場合、次のようになります。

```
+--------+--------+
|00000001|01110000|
+--------+--------+
```

When offsets and integer sizes are aligned to bytes boundaries, this is the
same as big endian, however when such alignment does not exist, its important
to also understand how the bits inside a byte are ordered.
